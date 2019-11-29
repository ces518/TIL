# @Async

spring 에서 제공하는 비동기 지원 어노테이션


# spring-boot

Application 클래스위에 @EnableAsync 어노테이션 설정시

SimpleAsyncTaskExecutor 가 비동기 처리스레드로 구동된다.

그후 비동기 처리를 원하는 클래스위에 @Async 어노테이션 설정시 비동기로 처리됨.

이 방법은 스레드 관리가 이뤄지지않음.

### Async Config Class (Thread pool 을 활용한 관리방법 )

Application 클래스의 @EnableAsync을 제거

Application 클래스에 @EnableAutoConfiguration(혹은 @SpringBootApplication) 설정이 되어있다면
런타임시 @Configuration가 설정된 SpringAsyncConfig 클래스의 threadPoolTaskExecutor bean 정보를 읽어들임.

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig {

    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(3);
        taskExecutor.setMaxPoolSize(30);
        taskExecutor.setQueueCapacity(10);
        taskExecutor.setThreadNamePrefix("Executor-");
        taskExecutor.initialize();
        return taskExecutor;
    }

}

@Async("threadPoolTaskExecutor")
public void method1() throws Exception {
    // do something
}

어노테이션에 스레드풀의 명을 달아주면 해당 스레드풀로 관리됨.

```

# 장점

method1에 대한 수정없이 처리가 가능하다는 점이 장점입
기존에는 동기/비동기에 따라서 method1의 내용이 달라짐
개발할 때 동기/비동기에 대한 고민 X
비동기로 처리해야되면 @Async annotation 사용

물론 Biz객체는 client객체에게 DI로 제공되어야만 합니다.
우리는 DI(Dependance Injection)만 신경쓰면 부수적인 것은 Spring이 다해줍니다.


# 제약사항
pulbic method에만 사용가능 합니다.
같은 객체내의 method끼리 호출시에는 @Async가 설정되어있어도 비동기처리가 되지 않습니다.

HttpServletRequest 와 Session에 접근 불가
이 사항은 Spring의 제약사항이 아니라 thread가 분리되는 비동기 처리이기 때문에 발생하는 현상입니다.
Spring @Async HttpServletRequest Session 글에서 @Async를 사용하여 비동기처리를 하면서도 Session 정보를 사용하는 방법을 설명드립니다.


# 예외처리

비동기로 처리되는 method에서 exception이 발생하면 어떻게 될까요?
해당 thread만 죽고 전체 프로세스에는 영향이 없습니다. 하지만 해당 thread가 소리없이 죽기 때문에 관리가 되지 않습니다.
Handler를 이용해서 exception에 대한 처리 방법을 알려드립니다.

그리고 WAS 종료시 threadPoolTaskExecutor가 정상적으로 종료되도록 destroyMethod을 설정했습니다.
해당 설정이 없으면 WAS 종료시 아래와 같은 경고 메시지를 만날 수 있습니다.

WAS 프로세스가 종료되는 과정에서 threadPoolTaskExecutor bean을 종료해야하는데 그 방법을 약속하지 않았고 결국 종료에 실패해서 발생하는 메시지 입니다. 이 상황에서는 memory leak과는 큰 상관없고 결국 JVM의 heap memory가 OS에게 반환되면서 threadPoolTaskExecutor bean이 차지하던 memory도 당연히 반환됩니다.

다만 WAS 종료시마다 경고 메시지가 뜨게 되고 이러한 불필요한 경고, 에러 메시지가 많이 발생하는 시스템은 운영자가 정작 중요한 경고, 에러 메시지를 놓치기 쉽기 때문에 메시지가 발생하지 않도록 처리하는 것이 좋습니다.
```java
public class HandlingExecutor implements AsyncTaskExecutor {
        private AsyncTaskExecutor executor;

        public HandlingExecutor(AsyncTaskExecutor executor) {
            this.executor = executor;
        }

        @Override
        public void execute(Runnable task) {
            executor.execute(createWrappedRunnable(task));
        }

        @Override
        public void execute(Runnable task, long startTimeout) {
            executor.execute(createWrappedRunnable(task), startTimeout);
        }

        @Override
        public Future<?> submit(Runnable task) {
            return executor.submit(createWrappedRunnable(task));
        }

        @Override
        public <T> Future<T> submit(final Callable<T> task) {
            return executor.submit(createCallable(task));
        }

        private <T> Callable<T> createCallable(final Callable<T> task) {
            return new Callable<T>() {
                @Override
                public T call() throws Exception {
                    try {
                        return task.call();
                    } catch (Exception ex) {
                        handle(ex);
                        throw ex;
                    }
                }
            };
        }

        private Runnable createWrappedRunnable(final Runnable task) {
            return new Runnable() {
                @Override
                public void run() {
                    try {
                        task.run();
                    } catch (Exception ex) {
                        handle(ex);
                    }
                }
            };
        }

        private void handle(Exception ex) {
            errorLogger.info("Failed to execute task. : {}", ex.getMessage());
            errorLogger.error("Failed to execute task. ",ex);
        }

        public void destroy() {
            if(executor instanceof ThreadPoolTaskExecutor){
                ((ThreadPoolTaskExecutor) executor).shutdown();
            }
        }
    }

}
```
