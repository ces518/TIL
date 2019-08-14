# Spring - Batch Start
- dependencies
    - devtools
    - lombok
    - jpa
    - mysql
    - h2
    - jdbc
    - batch

- Batch Job 을 생성하기 이전, Batch Configuration 활성화
```java
@EnableBatchProcessing
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }

}
```

- 간단한 Spring-Batch Code
```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class SimpleJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job simpleJob () {
        return jobBuilderFactory.get("simpleJob")
                .start(simpleStep1())
                .build();
    }

    @Bean
    public Step simpleStep1 () {
        return stepBuilderFactory.get("simpleStep1")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>> This is Step1");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

#### Job
- Spring-Batch 의 모든 Job은 Configuration class에 등록하여 사용한다.
- Job의 이름은 별도로 지정하지않고, Builder를 통해 지정한다.
- Spring Batch에서 Job은 하나의 배치 작업단위를 의미함
- Job내부에 여러 Step이 존재하며, Step내부에는 Tasklet , Reader & Processor, Writer 단위가 존재한다.
- Tasklet 과 Reader & Processor, Writer 는 동일한 LEVEL 이다.


#### Step
- Step 도 Job과 동일하게 Builder를 통해 이름을 지정한다.

#### Tasklet
- Step안에서 수행될 기능들을 명시한다.
- Step내에서 단일로 수행할 커스텀한 기능을 선언할때 사용함 


#### Meta Table
- Spring Batch를 사용하려면 Meta Table이 필요하다.
- Meta Table은 다음과 같은 내용을 가지고있다.
    - 이전에 실행한 Job에 대한 정보
    - 최근에 실패한 Batch Parameter가 어떤것이 존재하며 성공한 Job에 대한 정보
- 재 실행한다면 시작점에 대한 정보
- 어떤 Job에 어떤 Step이 존재하고, Step중 성공한 Step와 실패한 Step에 대한 정보
- 등등..
- H2 를 사용할경우 Boot가 자동 생성해주지만 상용 DB를 사용한다면 별도로 생성해 주어야한다.
- schema- 로 검색하면 DBMS에 맞게 스키마가 존재함.


#### BATCH_JOB_INSTANCE
- JOB_NAME
    - 수행한 BatchJobName
- Job Parameter에 따라 생성된다.
- Spring Bacth가 실행시 외부에서 받을수 있는 파라메터
- 특정 날짜를 Job Parameter로 넘기면 해당 날짜 데이터로 조회, 가공, 입력 등의 작업을 진행할 수있음.
- 같은 Batch Job 이라도 Job Parameter가 다르다면, 기록이 된다.
- 실행 옵션의 ProgramArguments로 전달 가능함.
- 동일한 Parameter로는 실행이 불가능하다.

- Job 코드 수정
```java
@Configuration
public class SimpleJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job simpleJob () {
        return jobBuilderFactory.get("simpleJob")
                .start(simpleStep1(null))
                .build();
    }

    @Bean
    @JobScope
    public Step simpleStep1 (@Value("#{jobParameters[requestDate]}") String requestDate) {
        return stepBuilderFactory.get("simpleStep1")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>> This is Step1");
                    log.info(">>>>> request Date = {}", requestDate);
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```


#### BATCH_JOB_EXECUTION
- JOB_INSTANCE 와 부모 - 자식 관계이다.
- JOB_INSTANCE의 성공/실패 내역을 가지고있음.

- JobConfiguration 변경
    - 실패하는 Job 기록
```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class SimpleJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job simpleJob () {
        return jobBuilderFactory.get("simpleJob")
                .start(simpleStep1(null))
                .next(simpleStep2(null))
                .build();
    }

    @Bean
    @JobScope
    public Step simpleStep1 (@Value("#{jobParameters[requestDate]}") String requestDate) {
        return stepBuilderFactory.get("simpleStep1")
                .tasklet((contribution, chunkContext) -> {
                    throw new IllegalArgumentException("step1 에서 실패");
                })
                .build();
    }

    @Bean
    @JobScope
    public Step simpleStep2 (@Value("#{jobParameters[requestDate]}") String requestDate) {
        return stepBuilderFactory.get("simpleStep2")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>> This is Step2");
                    log.info(">>>>> request Date = {}", requestDate);
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

- Exception 이 발생하며 Job이 실패함
- BATCH_JOB_EXECUTION 테이블에 FAILED 로 기록이 되며 원인 Exception 도 기록됨
```java
2019-08-14 16:41:51.500  INFO 16064 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=simpleJob]] completed with the following parameters: [{requestDate=20190815}] and the following status: [FAILED]
2019-08-14 16:41:51.506  INFO 16064 --- [      Thread-12] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2019-08-14 16:41:51.521  WARN 16064 --- [      Thread-12] o.s.b.f.support.DisposableBeanAdapter    : Invocation of destroy method failed on bean with name 'inMemoryDatabaseShutdownExecutor': java.sql.SQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'SHUTDOWN' at line 1
2019-08-14 16:41:51.521  INFO 16064 --- [      Thread-12] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2019-08-14 16:41:51.525  INFO 16064 --- [      Thread-12] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.

Process finished with exit code 0
```

- 다시 정상적인 Job 코드로 변경
- 성공한 JOB이 기록됨
- 같은 파라메터로 에러가 발생하지 않았음.
    - 동일한 Job Parameter로 성공 기록이 존재시에만 재수행이 되지않는다.
```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class SimpleJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job simpleJob () {
        return jobBuilderFactory.get("simpleJob")
                .start(simpleStep1(null))
                .next(simpleStep2(null))
                .build();
    }

    @Bean
    @JobScope
    public Step simpleStep1 (@Value("#{jobParameters[requestDate]}") String requestDate) {
        return stepBuilderFactory.get("simpleStep1")
                .tasklet((contribution, chunkContext) -> {
//                    throw new IllegalArgumentException("step1 에서 실패");
                    log.info(">>>>> This is Step1");
                    log.info(">>>>> request Date = {}", requestDate);
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    @JobScope
    public Step simpleStep2 (@Value("#{jobParameters[requestDate]}") String requestDate) {
        return stepBuilderFactory.get("simpleStep2")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>> This is Step2");
                    log.info(">>>>> request Date = {}", requestDate);
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```
