# Spring 5 in Action

## 15장 실패와 지연 처리하기

### 서킷 브레이커 패턴
- **서킷 브레이커 (Circuit Breaker)** 패턴은 우리가 작성한 코드가 실행에 실패하는 경우, 안전하게 처리되도록 도와준다.
- 이는 마이크로서비스의 컨텍스트에서 훨씬 더 중요하다.
- 전기 회로 차단기와 유사하다. 메소드의 호출을 허용하며, 닫힘상태에서 시작한다.
- 어떤 이유든, 메소드 실행이 실패할 경우 서킷 브레이커가 개방되고 실패한 메소드에 대해 더이상 호출이 되지 않는다.
- 전기 회로 차단기와는 조금 다른 점이 fallback 을 제공하여 자체적인 실패를 처리한다.
> https://github.com/ces518/TIL/blob/master/design-pattern/CircuitBreakerPattern.md

#### 서킷 브레이커, 어디에 적용해야 할까 ?
- 서킷 브레이커는 메소드에 적용된다.
- 따라서 하나의 마이크로서비스에는 많은 서킷브레이커가 있을 수 있다.
- 우리가 서킷브레이커를 선언할때는 **실패 대상이되는 메소드를 식별** 하는것이 중요하다.
  - REST 를 호출하는 메소드
  - 데이터베이스 쿼리를 수행하는 메소드
  - 느리게 실행될 가능성이 있는 메소드

> Netflix Hystrix 는 서킷 브레이커 패턴을 자바로 구현한 라이브러리 이다.
> 대상 메소드 실패시 폴백 메소드를 호출하는 Aspect 로 구현된다. (Proxy 사용)

### Spring Cloud Netflix Hystrix
- 서킷브레이커를 사용하려면 spring-cloud-starter-netflix-hystrix 의존성을 추가해야한다.

`pom.xml`
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

`Application.java`
```java
@SpringBootApplication
@EnableHystrix
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
- @EnableHystrix 애노테이션을 지정해서 Hystrix 를 활성화 한다.

`BookClient.java`
```java
@Component
public class BookClient {
    private RestTemplate restTemplate;

    public BookClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    // 장애 발생시 *베가스의 규칙* 을 적용해야 한다.
    // 마이크로서비스에서 생긴 에러는 다른 곳에 전파하지 않고, 해당 서비스내에 남겨야함
    @HystrixCommand(fallbackMethod = "getDefaultBooks") // fallbackMethod 를 지정하면, 서킷 브레이커 동작시 해당 메소드를 호출한다.
    public List<BookDto> findAll() {
        ResponseEntity<List<BookDto>> responseEntity = restTemplate.exchange("http://book-service/books", HttpMethod.GET, HttpEntity.EMPTY, new ParameterizedTypeReference<List<BookDto>>() {});
        List<BookDto> results = responseEntity.getBody();
        return results;
    }

    // Hystrix 의 기본 타임아웃은 1초이며, 대부분의 경우 적합하다.
    // 다음은 타임아웃을 0.5초로 설정하는 예제이다.
    @HystrixCommand(fallbackMethod = "getDefaultBook",
        commandProperties = {
                @HystrixProperty(
                        name = "execution.isolation.thread.timeoutInMilliseconds", // 서킷브레이커 타임아웃 설정, 기본값은 1초
                        value = "500"
                ),
                @HystrixProperty(
                        name = "execution.timeout.enabled", // 타임아웃이 필요없는 경우 사용하는 설정. 연쇄 지연효과가 발생할 수 있으므로 조심해야 한다.
                        value = "false"
                ),
                @HystrixProperty(
                        name = "circuitBreaker.requestVolumeThreshold", // 지정된 시간내에 메소드가 호출되어야 하는 횟수
                        value = "30"
                ),
                @HystrixProperty(
                        name = "circuitBreaker.errorThresholdPercentage", // 지정된 시간내에 실패한 메소드 호출 비율, 50%
                        value = "25"
                ),
                @HystrixProperty(
                        name = "metrics.rollingStats.timeInMilliseconds", // 요청 횟수와 에러 비율이 고려되는 시간
                        value = "20000"
                ),
                @HystrixProperty(
                        name = "circuitBreaker.sleepWindowInMilliseconds", // 절반-열림 상태로 진입하여 실패한 메소드가 다시 시도되기 전 열림 상태서킷이 유지되는 시간
                        value = "20000"
                )
        }
    )
    // [기본 설정]
    // 10초동안 20번이상 호출되고, 50%이상이 실패한다면 서킷 브레이커가 동작하여 열림 상태가 된다.
    // 5초 후 절반-열림 상태가 되어 기존 메소드 호출이 다시 시도된다.
    // 위 설정은 20초동안 메소드가 30번이상 호출되어 25%이상 실패할 경우 서킷브레이커가 동작하도록 설정한 예제이다.
    public BookDto findById(Long bookId) {
        return restTemplate.getForEntity("http://book-serivce/books/{bookId}", BookDto.class, bookId).getBody();
    }

    // fallbackMethod 에도 서킷브레이커 지정이 가능하다.
    // @HystrixCommand 를 사용하여 연쇄적인 폴백 메소드들을 지정할 수 있다.
    private List<BookDto> getDefaultBooks() {
        return List.of(new BookDto(-1L, "제목", "작성자", "설명"));
    }

    private BookDto getDefaultBook(Long bookId) {
        return new BookDto(-1L, "제목", "작성자", "설명");
    }
}
```
- @HystrixCommand 애노테이션을 지정하면 서킷 브레이커가 적용된다.
  - fallbackMethod 속성을 이용해, fallback 처리를 수행할 메소드를 지정해준다.
  - commandProperties 속성을 이용해 다양한 Hystrix 관련설정을 할 수 있다.
  - 간단하게 몇가지만 살펴보면 다음과 같다.
    1. execution.isolation.thread.timeoutInMilliseconds : 서킷브레이커 타임아웃 설정이며 기본값은 1초이다.
    2. execution.timeout.enabled : 타임아웃이 필요없는 경우 지정해주어야 한다. 기본값은 true
    3. circuitBreaker.requestVolumeThreshold : 지정된 시간내에 메소드가 호출되어야 하는 횟수
- https://github.com/Netflix/Hystrix/wiki/Configuration

> 주의할 점은 fallback 메소드 지정시, 메소드 시그니쳐가 동일해야 한다.
> 또한 fallback 메소드가 실패할 경우도 있으므로, fallback 메소드에도 Hystrix 를 적용할 수 있으며, 연쇄적인 폴백처리가 가능하다.

### 실패 모니터링하기
- 서킷 브레이커로 보호되는 메소드는 매번 호출시마다 해당 호출에 관한 데이터가 수집되어 Hystrix 스트림으로 발행된다.
- 이 스트림은 실행중인 애플리케이션의 건강 상태를 실시간으로 모니터링 하는데 사용가능하다.
- 이 스트림이 포함하는 데이터는 다음과 같다.
  1. 메소드의 호출 횟수
  2. 성공적인 호출 횟수
  3. 폴백 메소드의 호출 횟수
  4. 메소드의 타임아웃 횟수
- Hystrix 스트림은 Actuator endpoint 로 제공되기 때문에 각 서비스마다 actuator 의존성을 추가해 주어야한다.

`pom.xml`
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

`Application.yaml`
```yaml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```
> Hystrix 스트림 엔드포인트는 /actuator/hystrix.stream 인데, 대부분의 엔드포인트는 비활성화 되어 있다.
> 위와 같이 활성화시켜 주어야한다.

### Hystrix 대시보드
- Hystrix 스트림은 온갖 json 데이터가 가득 차 있으므로, 클라이언트 측에서 데이터 해석을 위한 작업이 많이 필요하다.
- 이럴 경우 Hystrix 대시보드의 사용을 고려할 수 있다.
- Hystrix 대시보드를 사용하려면 spring-cloud-starter-netflix-hystrix-dashboard 의존성을 추가해 주어야한다.

`pom.xml`
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

`Application.java`
```java
@SpringBootApplication
@EnableHystrixDashboard
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
- @EnableHystrixDashboard 애노테이션을 사용해서 자동구성을 하도록 명시한다.

### Hystrix 의 스레드풀
- 특정 메소드가 해당 요청을 수행하는데 시간이 오래 걸린다면, 해당 서비스가 응답할 때까지 Hystrix 는 응답을 대기하며 관련 스레드를 **블로킹** 한다.
- 만약 해당 스레드가 호출자와 같은 스레드의 컨텍스트에서 실행중이라면, 호출자는 오랫동안 실행되는 메소드로부터 벗어날 수 없다.
- 블로킹된 스레드가 요청 제한된 수의 스레드 (톰캣의 요청처리를 수행하는 스레드 등) 이라면 결국 스레드 고갈이 나게 됨
- 이를 방지하기 위해 Hystrix 는 각 의존성 모듈의 스레드풀을 할당한다. (별도의 스레드풀을 사용한다!! 중요)
- Hystrix 메소드 중 하나 호출시 이 스레드풀의 스레드를 사용한다.
- 스레드 풀의 대안으로 **semaphore isolation** 을 사용할 수 있다.
> 기본적으로 @HystrixCommand 는 별도의 Thread 로 동작하기 때문에, ThreadLocal 이나 Spring 의 @RequestScope, @SessionScope 빈에 접근이 불가능 하다.
> execution.isolation.strategy: SEMAPHORE 로 지정하여 동일 스레드에서 연산을 지정하도록 실행수 있다.
> hystrix.shareSecurityContext=true 를 사용해서 스프링 시큐리티 컨텍스트도 공유할 수 있다.
- https://supawer0728.github.io/2018/03/11/Spring-Cloud-Hystrix/

### 정리
- 서킷 브레이커 패턴은 유연한 실패 처리를 할 수 있다.
- Hystrix 는 서킷 브레이커 패턴을 구현한다.
- Hystrix 가 제공하는 서킷브레이커는 애플리케이션의 상태를 체크하는 목적으로 Hystrix 스트림을 발행한다.
- Hystrix 가 발행하는 스트림을 수집해서 Hystrix 대시보드로 시각화 할 수 있다.