# Spring 5 in Action

## 9장 스프링 통합하기
- 스프링 통합은 <Enterprise Integration Patterns> 에서 보여준 대부분의 통합 패턴을 사용할수 있게 구현한것이다.

### 간단한 플로우 선언하기
- 통합 플로우를 통해 외부 리로스 혹은 애플리케이션에 데이터를 수신 및 전송을 할 수 있다.
- 그중 하나가 파일 시스템이다.
- 파일 시스템에 데이터를 쓰는 통합 플로우를 사용하기 위해 아래 의존성을 추가해 주자.

```xml
<!-- 스프링 통합의 부트 스타터 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-integration</artifactId>
</dependency>

<!-- 스프링 통합의 파일 엔드포인트 모듈 -->
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-file</artifactId>
</dependency>
```

- 파일에 데이터를 쓰려면 통합 플로우로 데이터를 전송하는 **게이트웨이 (Gateway)** 를 생성해야 한다.

`FileWriterGateway`
```java
// 메시지 게이트웨이 선언
@MessagingGateway(defaultRequestChannel = "textInChannel")
public interface FileWriterGateway {
    void writeToFile(@Header(FileHeaders.FILENAME) String filename, String data);
}
```
- @MessagingGateway 애노테이션으로 게이트웨이임을 명시하면, 런타임 시점에 스프링 통합이 실제 구현체를 생성해준다.
- defaultRequestChannel 은 해당 인터페이스로 메서드 호출로 생성된 메시지가 지정된 채널로 전송된다. (textInChannel)

- 통합 플로우는 XML, Java, DSL 세 가지 방법으로 정의할 수 있다.
- Java 와 DSL 구성만 우선적으로 살펴볼 것이다.

### Java 로 통합 플로우 구성하기
```java
@Configuration
public class FileWriterIntegrationConfig {

  @Bean
  @Transformer(inputChannel = "textInChannel", outputChannel = "fileWriterChannel") // 변환기 선언
  public GenericTransformer<String, String> upperCaseTransformer() {
    return text -> text.toUpperCase();
  }

  @Bean
  @ServiceActivator(inputChannel = "fileWriterChannel") // 파일쓰기 핸들러 선언
  public FileWritingMessageHandler fileWriter() {
    FileWritingMessageHandler handler = new FileWritingMessageHandler(new File("/tmp/sia5/files"));
    handler.setExpectReply(false); // false 로 지정하지 않을경우, 정상 동작하더라도 응답 채널이 구성되지 않았다는 로그가 찍힌다.
    handler.setFileExistsMode(FileExistsMode.APPEND);
    handler.setAppendNewLine(true);
    return handler;
  }
}
```
- 자바 구성에서는 채널들을 별도로 지정하지 않아도 자동으로 생성해 준다.
- 하지만 명시적으로 선언해주고 싶다면 아래와 같이 설정해 주어야 한다.
```java
@Bean
public MessageChannel textInChannel() {
    return new DirectChannel();
}
```

### DSL 구성 사용하기
```java
@Bean
public IntegrationFlow fileWriterFlow() {
    return IntegrationFlows.from(MessageChannels.direct("textInChannel"))
            .<String, String>transform(t -> t.toUpperCase())
            .channel(MessageChannels.direct("fileWriterChannel")) // 별도 채널을 구성하지 않아도 스프링이 자동 생성해주지만, 필요한 경우 선언할 수 있다.
            .handle(Files
                    .outboundAdapter(new File("/tmp/sia5/files"))
                    .fileExistsMode(FileExistsMode.APPEND)
                    .appendNewLine(true)
            ).get();
}
```

### 스프링 통합 컴포넌트 살펴보기
- 채널 : 한 요소로 부터 다른 요소로 메시지 전달
- 필터 : 조건에 맞는 메시지가 플로우를 통과 하게 해준다
- 변환기 : 메시지 값을 변경하거나 페이로드의 타입을 다른 타입으로 변환
- 라우터 : 여러 채널 중 하나로 메시지를 전달하며, 대개 메시지 헤더를 기반으로 한다
- 분배기 : 들어오는 메시지를 두 개 이상의 메시지로 분할하며, 각 메시지는 다른 채널로 전송
- 집적기 : 분배기와 상반된 것으로 별개의 채널로 부터 전달되는 다수의 메시지를 하나의 메시지로 결합
- 액티베이터 : 메시지를 처리하도록 자바 메서드에 메시지를 넘긴 뒤 메서드의 반환 값을 출력 채널로 전송
- 채널 어댑터 : 외부 시스템에 채널을 연동한다.
- 게이트웨이 : 인터페이스를 통해 통합 플로우로 데이터를 전달한다.