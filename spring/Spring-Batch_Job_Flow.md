# Spring Batch Job Flow
- Job 을 구성하는데 Step이 존재함
- Step 은 실제 Batch 작업을 수행하는 역할
- Batch로 실제 처리하고자 하는 기능과 설정을 모두 포함
- Step들 간의 순서 혹은 처리흐름 제어가 필요함.

#### Next
- 순차적으로 Step들을 연결할때 사용 됨
- step1 > step2 > step3 와 같이 하나씩 실행시킬때 좋은 방법이다.
```java
@Slf4j
@Configuration
@RequiredArgsConstructor
public class StepNextJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job stepNextJob () {
        return jobBuilderFactory.get("stepNextJob")
                .start(step1())
                .next(step2())
                .next(step3())
                .build();
    }

    @Bean
    public Step step1 () {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                   log.info(">>>> This is Step1");
                   return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step step2 () {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>> This is Step2");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step step3 () {
        return stepBuilderFactory.get("step3")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>> This is Step3");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

- 실행 결과
    - stepNextJob 에서 정의한 Step 순서대로 실행이 된다. 
    - 방금 생성한 stepNextJob 뿐만 아니라 기존에 있떤 simpleJob도 실행 되었다.
```java
2019-08-16 09:49:19.003  INFO 11796 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=simpleJob]] launched with the following parameters: [{requestDate=20190816}]
2019-08-16 09:49:19.140  INFO 11796 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [simpleStep1]
2019-08-16 09:49:19.212  INFO 11796 --- [  restartedMain] m.june.bacth.job.SimpleJobConfiguration  : >>>>> This is Step1
2019-08-16 09:49:19.213  INFO 11796 --- [  restartedMain] m.june.bacth.job.SimpleJobConfiguration  : >>>>> request Date = 20190816
2019-08-16 09:49:19.400  INFO 11796 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [simpleStep2]
2019-08-16 09:49:19.467  INFO 11796 --- [  restartedMain] m.june.bacth.job.SimpleJobConfiguration  : >>>>> This is Step2
2019-08-16 09:49:19.468  INFO 11796 --- [  restartedMain] m.june.bacth.job.SimpleJobConfiguration  : >>>>> request Date = 20190816
2019-08-16 09:49:19.627  INFO 11796 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=simpleJob]] completed with the following parameters: [{requestDate=20190816}] and the following status: [COMPLETED]
2019-08-16 09:49:19.732  INFO 11796 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=stepNextJob]] launched with the following parameters: [{requestDate=20190816}]
2019-08-16 09:49:19.847  INFO 11796 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step1]
2019-08-16 09:49:19.917  INFO 11796 --- [  restartedMain] m.j.bacth.job.StepNextJobConfiguration   : >>>> This is Step1
2019-08-16 09:49:20.145  INFO 11796 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step2]
2019-08-16 09:49:20.217  INFO 11796 --- [  restartedMain] m.j.bacth.job.StepNextJobConfiguration   : >>>> This is Step2
2019-08-16 09:49:20.424  INFO 11796 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step3]
2019-08-16 09:49:20.491  INFO 11796 --- [  restartedMain] m.j.bacth.job.StepNextJobConfiguration   : >>>> This is Step3
2019-08-16 09:49:20.658  INFO 11796 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=stepNextJob]] completed with the following parameters: [{requestDate=20190816}] and the following status: [COMPLETED]
```

#### 지정한 Batch Job 만 실행
- Spring Batch 실행시 Program arguments 로 값이 넘어오면 해당 값과 일치하는 Job만 실행 하도록 설정을 변경
- ${job.name:NONE}
    - job.name 값이 존재하면 해당 값을 할당하고, 없다면 할당하지 않겠다는 의미
    - 값이 없다면 모든 배치가 실행하지 않도록 막는 역할
- application.yml
```yml
spring:
  profiles:
    active: local
  batch:
    job:
      names: ${job.name:NONE}
```

- ProgramArgument 를 변경해서 Batch 실행
    - --job.name=stepNextJob version=2
- stepNextJob 만 실행 되는걸 확인 가능
```java
2019-08-16 10:10:06.546  INFO 24920 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=stepNextJob]] launched with the following parameters: [{version=2, -job.name=stepNextJob}]
2019-08-16 10:10:06.670  INFO 24920 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step1]
2019-08-16 10:10:06.757  INFO 24920 --- [  restartedMain] m.j.bacth.job.StepNextJobConfiguration   : >>>> This is Step1
2019-08-16 10:10:06.969  INFO 24920 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step2]
2019-08-16 10:10:07.054  INFO 24920 --- [  restartedMain] m.j.bacth.job.StepNextJobConfiguration   : >>>> This is Step2
2019-08-16 10:10:07.321  INFO 24920 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step3]
2019-08-16 10:10:07.422  INFO 24920 --- [  restartedMain] m.j.bacth.job.StepNextJobConfiguration   : >>>> This is Step3
2019-08-16 10:10:07.741  INFO 24920 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=stepNextJob]] completed with the following parameters: [{version=2, -job.name=stepNextJob}] and the following status: [COMPLETED]
```

#### 흐름제어 Flow
- next 를 사용하면 Step의 순서를 제어 한다
- 하지만 문제점은 이전의 Step에서 오류가 발생하면 나머지 Step들은 실행되지 못한다.
- 상황에 따라 정상일때는 Step B, 오류시 Step C 를 수행해야하는 경우가 있다면 ?
    - 이런 경우 Spring Batch Job 에서는 조건별로 Step을 사용할 수 있음.

- Step1의 실패 여부에 따라 시나리오가 달라진다.
    - step1 실패: step1 -> step3
    - step1 성공: step1 -> step2 -> step3

- stepNextConfigurationJob 이 전체 Flow 를 관리한다.
    - on()
        - Catch 할 ExitStatus 를 지정
        - * 일경우 모든 ExitStatus
    - to()
        - 다음으로 이동할 Step 을 지정
    - from()
        - 이벤트 리스너 역할
        - 상태 값을 보고 일치하는 상태일 경우 to() 에 지정한 step을 호출한다.
        - step1의 이벤트 캐치가 FAILED 라면 추가 이벤트 캐치시 from 을 써야한다.
    - end()
        - FlowBuilder 를 반환하는 end()
            - on()뒤에 체이닝 하는 end()
        - FlowBuilder 를 종료하는 end()
            - build() 이전의 end()

##### on() 이 캐치하는 상태값은 ExitStatus 이다
##### 분기처리를 위해 상태값 조정이 필요하다면 ExitStatus 를 조정해야한다.
```java
@Slf4j
@Configuration
@RequiredArgsConstructor
public class StepNextConditionalJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job stepNextConfigurationJob () {
        return jobBuilderFactory.get("stepNextConfigurationJob")
                .start(conditionalJobStep1())
                    .on("FAILED") // 실패시
                    .to(conditionalJobStep3()) // step3
                    .on("*") // 결과 무관
                    .end() // step3 종료시 Flow 종료
                .from(conditionalJobStep1())
                    .on("*") // FAILED 외 모든 경우
                    .to(conditionalJobStep2()) // step2
                    .next(conditionalJobStep3()) //step2 정상 종료시 step3
                    .on("*") // step3 결과 무관
                    .end() // Flow 종료
                .end() // job 종료
                .build();
    }

    @Bean
    public Step conditionalJobStep1 () {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>>>> this is step 1");
                    /**
                        ExitStatus 를 FAILED 로 지정
                    **/
                    contribution.setExitStatus(ExitStatus.FAILED);
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step conditionalJobStep2 () {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>>>> this is step 2");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step conditionalJobStep3 () {
        return stepBuilderFactory.get("step3")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>>>> this is step 3");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

}
```

#### FAILED FLOW 실행 결과
```java
2019-08-16 11:25:02.271  INFO 26232 --- [  restartedMain] o.s.b.a.b.JobLauncherCommandLineRunner   : Running default command line with: [--job.name=stepNextConfigurationJob, version=3]
2019-08-16 11:25:02.555  INFO 26232 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=stepNextConfigurationJob]] launched with the following parameters: [{version=3, -job.name=stepNextConfigurationJob}]
2019-08-16 11:25:02.685  INFO 26232 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step1]
2019-08-16 11:25:02.747  INFO 26232 --- [  restartedMain] .b.j.StepNextConditionalJobConfiguration : >>>>>>> this is step 1
2019-08-16 11:25:02.969  INFO 26232 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step3]
2019-08-16 11:25:03.028  INFO 26232 --- [  restartedMain] .b.j.StepNextConditionalJobConfiguration : >>>>>>> this is step 3
```

#### 성공 FLOW 실행 결과
```java
2019-08-16 11:26:35.534  INFO 7160 --- [  restartedMain] o.s.b.a.b.JobLauncherCommandLineRunner   : Running default command line with: [--job.name=stepNextConfigurationJob, version=4]
2019-08-16 11:26:35.842  INFO 7160 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=stepNextConfigurationJob]] launched with the following parameters: [{version=4, -job.name=stepNextConfigurationJob}]
2019-08-16 11:26:36.020  INFO 7160 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step1]
2019-08-16 11:26:36.098  INFO 7160 --- [  restartedMain] .b.j.StepNextConditionalJobConfiguration : >>>>>>> this is step 1
2019-08-16 11:26:36.328  INFO 7160 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step2]
2019-08-16 11:26:36.397  INFO 7160 --- [  restartedMain] .b.j.StepNextConditionalJobConfiguration : >>>>>>> this is step 2
2019-08-16 11:26:36.628  INFO 7160 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step3]
2019-08-16 11:26:36.688  INFO 7160 --- [  restartedMain] .b.j.StepNextConditionalJobConfiguration : >>>>>>> this is step 3
```

#### Batch Status 와 Exit Status 의 차이
##### Batch Status
- Job or Step 의 실행 결과를 기록할때 사용하는 Enum
- on("FAILED") 에서 참조하는것은 ExitStatus 이다.
```java
package javax.batch.runtime;

/**
 * BatchStatus enum defines the batch status values
 * possible for a job.
 *
 */
public enum BatchStatus {STARTING, STARTED, STOPPING, 
			STOPPED, FAILED, COMPLETED, ABANDONED }
```

##### Exit Status
- Step 실행후 의 상태를 의미
- Enum 이 아니다.
- Spring Batch 는 기본적으로 ExitStatus 의 exitCode는 Step의 BatchStatus와 동일하도록 설정됨
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.batch.core;

import java.io.PrintWriter;
import java.io.Serializable;
import java.io.StringWriter;
import org.springframework.util.StringUtils;

public class ExitStatus implements Serializable, Comparable<ExitStatus> {
    public static final ExitStatus UNKNOWN = new ExitStatus("UNKNOWN");
    public static final ExitStatus EXECUTING = new ExitStatus("EXECUTING");
    public static final ExitStatus COMPLETED = new ExitStatus("COMPLETED");
    public static final ExitStatus NOOP = new ExitStatus("NOOP");
    public static final ExitStatus FAILED = new ExitStatus("FAILED");
    public static final ExitStatus STOPPED = new ExitStatus("STOPPED");
    private final String exitCode;
    private final String exitDescription;

    public ExitStatus(String exitCode) {
        this(exitCode, "");
    }

    public ExitStatus(String exitCode, String exitDescription) {
        this.exitCode = exitCode;
        this.exitDescription = exitDescription == null ? "" : exitDescription;
    }

    public String getExitCode() {
        return this.exitCode;
    }

    public String getExitDescription() {
        return this.exitDescription;
    }

    public ExitStatus and(ExitStatus status) {
        if (status == null) {
            return this;
        } else {
            ExitStatus result = this.addExitDescription(status.exitDescription);
            if (this.compareTo(status) < 0) {
                result = result.replaceExitCode(status.exitCode);
            }

            return result;
        }
    }

    public int compareTo(ExitStatus status) {
        if (this.severity(status) > this.severity(this)) {
            return -1;
        } else {
            return this.severity(status) < this.severity(this) ? 1 : this.getExitCode().compareTo(status.getExitCode());
        }
    }

    private int severity(ExitStatus status) {
        if (status.exitCode.startsWith(EXECUTING.exitCode)) {
            return 1;
        } else if (status.exitCode.startsWith(COMPLETED.exitCode)) {
            return 2;
        } else if (status.exitCode.startsWith(NOOP.exitCode)) {
            return 3;
        } else if (status.exitCode.startsWith(STOPPED.exitCode)) {
            return 4;
        } else if (status.exitCode.startsWith(FAILED.exitCode)) {
            return 5;
        } else {
            return status.exitCode.startsWith(UNKNOWN.exitCode) ? 6 : 7;
        }
    }

    public String toString() {
        return String.format("exitCode=%s;exitDescription=%s", this.exitCode, this.exitDescription);
    }

    public boolean equals(Object obj) {
        return obj == null ? false : this.toString().equals(obj.toString());
    }

    public int hashCode() {
        return this.toString().hashCode();
    }

    public ExitStatus replaceExitCode(String code) {
        return new ExitStatus(code, this.exitDescription);
    }

    public boolean isRunning() {
        return "EXECUTING".equals(this.exitCode) || "UNKNOWN".equals(this.exitCode);
    }

    public ExitStatus addExitDescription(String description) {
        StringBuilder buffer = new StringBuilder();
        boolean changed = StringUtils.hasText(description) && !this.exitDescription.equals(description);
        if (StringUtils.hasText(this.exitDescription)) {
            buffer.append(this.exitDescription);
            if (changed) {
                buffer.append("; ");
            }
        }

        if (changed) {
            buffer.append(description);
        }

        return new ExitStatus(this.exitCode, buffer.toString());
    }

    public ExitStatus addExitDescription(Throwable throwable) {
        StringWriter writer = new StringWriter();
        throwable.printStackTrace(new PrintWriter(writer));
        String message = writer.toString();
        return this.addExitDescription(message);
    }

    public static boolean isNonDefaultExitStatus(ExitStatus status) {
        return status == null || status.getExitCode() == null || status.getExitCode().equals(COMPLETED.getExitCode()) || status.getExitCode().equals(EXECUTING.getExitCode()) || status.getExitCode().equals(FAILED.getExitCode()) || status.getExitCode().equals(NOOP.getExitCode()) || status.getExitCode().equals(STOPPED.getExitCode()) || status.getExitCode().equals(UNKNOWN.getExitCode());
    }
}
```

#### 커스텀한 exitCode
- Spring Batch는 기본적으로 ExitStatus의 exitCode는 Step의 BatchStatus 와 같다고 했다.
- 커스텀 한 exitCode가 필요한경우 
    - step1이 종료시 COMPLETED WITH SKIPS 의 exitCode로 종료 된다.
    - 하지만 COMPLETED WITH SKIPS 는 ExitStatus 에는 존재하지 않는 코드
    - 원하는 대로 처리하기 위해서는 COMPLETED WITH SKIPS exitCode를 반환하는 로직이 필요함.
```java
.start(step1())
    .on("FAILED")
    .end()
.from(step1())
    .on("COMPLETED WITH SKIPS")
    .to(errorPrint1())
    .end()
.from(step1())
    .on("*")
    .to(step2())
    .end()
```

- SkipCheckingListener 에서는 Step이 성공적으로 수행 되었는지 확인후 skip횟수가 0보다 클경우 COMPLETED WITH SKIPS 의 exitCode를 가지는 ExitStatus 를 반환한다.
```java
public class SkipCheckingListener extends StepExecutionListenerSupport {

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        String exitCode = stepExecution.getExitStatus().getExitCode();
        // FAILED가 아닌경우
        if (!exitCode.equals(ExitStatus.FAILED.getExitCode()) &&
            stepExecution.getSkipCount() > 0) {
            return new ExitStatus("COMPLETED WITH SKIPS");
        } else {
            return null;
        }
    }
}
```

#### JobExecutionDecider 
- 이전 코드들의 문제점
    - Step이 담당하는 역할이 둘 이상이다.
        -  Step 이 처리해야할 로직 외 분기 처리를 하기위한 ExitStatus 조작 필요
    - 다양한 분기 로직 처리가 힘듦
        - ExitStatus 를 커스텀 하기위해 Listener를 생성하고 Job Flow에 등록하는 등 번거로움 존재
- Spring Batch 에는 Step 의 Flow 중 분기만 담당하는 Type 을 제공한다.
    - JobExecutionDecider

- Flow
    - startStep > decider 에서 RandomNumber를 생성, 홀수/짝수 구분 oddStep or evenStep 진행
```java
@Slf4j
@Configuration
@RequiredArgsConstructor
public class DeciderJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job jobWithDecider () {
        return jobBuilderFactory.get("jobWithDecider")
                .start(startStep())
                .next(decider())
                .from(decider())
                    .on("ODD")
                    .to(oddStep())
                .from(decider())
                    .on("EVEN")
                    .to(evenStep())
                .end()
                .build();
    }

    @Bean
    public Step startStep () {
        return stepBuilderFactory.get("startStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("start Step");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step evenStep () {
        return stepBuilderFactory.get("evenStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("even Step");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step oddStep () {
        return stepBuilderFactory.get("oddStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("odd Step");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public JobExecutionDecider decider () {
        return new JobExecutionDecider() {
            @Override
            public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
                Random random = new Random();

                int randomNumber = random.nextInt(50) + 1;
                log.info("randomNumber = {}", randomNumber);

                if (randomNumber % 2 == 0) {
                    return new FlowExecutionStatus("EVEN");
                }
                return new FlowExecutionStatus("ODD");
            }
        };
    }
}
```

- 분기와 관련된 로직은 JobExecutionDecider 의 구현체가 담당하고있다.
- Step과 역할과 책임이 분리 되어있음.
```java
@Bean
public JobExecutionDecider decider () {
    return new JobExecutionDecider() {
        @Override
        public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
            Random random = new Random();

            int randomNumber = random.nextInt(50) + 1;
            log.info("randomNumber = {}", randomNumber);

            if (randomNumber % 2 == 0) {
                return new FlowExecutionStatus("EVEN");
            }
            return new FlowExecutionStatus("ODD");
        }
    };
}
```

- 실행결과
```java
2019-08-16 17:47:20.209  INFO 13528 --- [  restartedMain] o.s.b.a.b.JobLauncherCommandLineRunner   : Running default command line with: [--job.name=jobWithDecider, version=7]
2019-08-16 17:47:20.489  INFO 13528 --- [  restartedMain] o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=jobWithDecider]] launched with the following parameters: [{version=7, -job.name=jobWithDecider}]
2019-08-16 17:47:20.640  INFO 13528 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [startStep]
2019-08-16 17:47:20.721  INFO 13528 --- [  restartedMain] m.j.bacth.job.DeciderJobConfiguration    : start Step
2019-08-16 17:47:20.857  INFO 13528 --- [  restartedMain] m.j.bacth.job.DeciderJobConfiguration    : randomNumber = 28
2019-08-16 17:47:20.940  INFO 13528 --- [  restartedMain] o.s.batch.core.job.SimpleStepHandler     : Executing step: [evenStep]
2019-08-16 17:47:21.007  INFO 13528 --- [  restartedMain] m.j.bacth.job.DeciderJobConfiguration    : even Step
```
