# ParallelStream vs CompletableFuture

기본 적으로 퍼포먼스는 ParallelStream이 좀더 빠르나, CompletableFuture의 ThreadPool을 조정하면 퍼포먼스가 크게 향상됨



#### ParallelStream

- java8에서 추가된 병렬처리를 매우 쉽게 해주는 방식
- ForkJoinFramework를 이용하여 작업들을 분할하고, 병렬적으로 처리하게 됨
- 가독성이 매우 좋다.



`기본 사용 방법`

````java
LongStream.range(0, 1_000_000_000).parallel() .sum();
````



> parallelStream은 개발자 모르게 내부 스레드풀을 만들어 작업을 하지만, Stream별도 스레드 풀을 만드는게 아닌 하나의 스레드 풀을 모든 parallelStream이 공유한다. 장애발생 요지가 있으니 주의해서 사용해야한다.





`스레드 4개짜리 별도의 스트림을 생성하게끔 해주는 코드`

```java
ForkJoinPool pool = new ForkJoinPool(4); 
long sum = pool.submit(() -> LongStream.range(0, 1_000_000_000).parallel() .sum()).get();
```



> ParallelStream 을 사용하면 발생하는 문제를 우회하는 방법이 있지만, 좋은 패턴은 아니다.



#### CompletableFuture

- Java 5 부터 미래의 어느시점에 결과를 얻는 모델에 활용할수 있도록 Future인터페이스를 제공하고 있다.
- 비동기 처리를 하는데 Future를 사용하며 Future는 해당 처리가 끝난뒤 결과를 얻을 수 있는 레퍼런스를 제공한다.
- CompletableFuture는 일반적으로 사용했을때는 parallelStream과 비슷하거나 성능이 떨어진다.
- 이를 개선하는 방법의 별도의 ThreadPool을 사용하는것이다.



`ThreadPool 등록`

```java
@Configuration
public class ThreadPoolConfig {

    @Bean("threadPool")
    public Executor threadPoolTaskExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(4); // 기본 스레드 수
        taskExecutor.setMaxPoolSize(8); // 최대 스레드 수
        taskExecutor.setQueueCapacity(100); // QUEUE 수
        taskExecutor.setThreadNamePrefix("custom-thread");
        taskExecutor.initialize();
        return taskExecutor;
    }
}
```



> ThreadPool 생성시 기본 Thread의 수를 지정해주어야한다.
>
> 일반적으로 CPU 연산작업은 코어의 수에 맞추는경우가 많다.
>
> 하지만 API CALL과 같이 스레드 반환하기 까지의 시간이 걸릴것으로 예상되는 경우에는 보다 넉넉하게 스레드 수를 지정해 주어야한다.
>
> 최악의 경우 심각한 장애까지 이어질 수 있다.



##### 사용방법

- 사용방법은 크게 2가지가 있다.
- 첫번째는 CompletableFuture.allOf(), 다른 하나는 stream()을 활용한 호출방법이다.



`CompletableFuture.allOf()`

````java
List<CompletableFuture<List<String>>> futures = lists.stream()
  .map(v -> CompletableFuture.supplyAsync(() -> donSomething(v), threadPool))
  .collect(Collectors.toList());

try {
  return CompletableFuture.allOf(futures.toArray(new CompletableFuture[futures.size()]))
    .thenApply(result ->
               futures.stream()
               .map(CompletableFuture::join)
               .flatMap(Collection::stream)
               .collect(Collectors.toList())
              )
    .get();
} catch (ExecutionException | InterruptedException e) {
  log.error("combined future exception", e.getMessage(), e);
  throw new RuntimeException(e);
}
````



> allOf()를 사용하는 방식의 단점은 get()을 호출했을때 ChekcedException이 발생하고, 코드가 장황해질 수 있다는 점이다.



`thenApply() vs thenApplyAsync()`

- 두가지 모두 비동기 처리에 대한 응답을 받는 메서드이다.
- 차이점을 간단하게 비교하자면 thenApply()는 작업을 수행할때 사용했던 스레드를 사용한다.
- ~~Async()가 붙어있는 메서드는 작업을 수행할때 사용했던 스레드가 아닌 다른 스레드를 사용한다.
  - 기본적으로 Common-Pool에 존재하는 스레드를 사용한다.

> 스레드가 블록 되지 않는 병렬처리를 원한다면 ~~Async()가 붙어있는 메서드를 사용할것을 권장한다.





`Stream을 활용한 방법`

````java
List<CompletableFuture<List<String>>> futures = lists.stream()
  .map(v -> CompletableFuture.supplyAsync(() -> donSomething(v), threadPool))
  .collect(Collectors.toList());

return futures.stream()
  .map(CompletableFuture::join)
  .flatMap(Collection::stream)
  .collect(Collectors.toList());
````



> Stream을 사용해서 join()을 호출하는 방식은 get()과 비슷하게 동작하지만, 예외가 발생했을때 uncheckedException이 발생한다.



##### 주의점

- 간혹가다 Stream을 활용하는 방법 사용시 두개의 스트림이 아닌 하나의 스트림으로 처리하는 경우가 있는데
- 이럴 경우 비동기 처리가아닌 동기식으로 처리가 된다.



````java
return lists.stream()
  .map(v -> CompletableFuture.supplyAsync(() -> donSomething(v), threadPool))
  .map(CompletableFuture::join)
  .flatMap(Collection::stream)
  .collect(Collectors.toList());
````



> 위 코드를 살펴보면 CompletableFuture.supplyAsync()를 사용해서 요청을 보냄과 동시에 CompletableFuture::join을 사용해서 응답을 기다리기 때문에 당연히 하나씩 처리를하고 다음 Future로 넘어가게 된다.




#### 참고

- https://multifrontgarden.tistory.com/254
- http://fahdshariff.blogspot.com/2016/06/java-8-completablefuture-vs-parallel.html
- https://stackoverflow.com/questions/58700578/why-is-completablefuture-join-get-faster-in-separate-streams-than-using-one-stre
- https://www.baeldung.com/java-completablefuture
- https://stackoverflow.com/questions/47489338/what-is-the-difference-between-thenapply-and-thenapplyasync-of-java-completablef/47489654
