# Log4j 와 Logback 

## Log4j
**Log4j (Log for java)** 는 현재 Apache Logging Service 이며 특징은 다음과 같다.

### 특징
1. 속도 및 유연성을 고려한 디자인, 속도에 최적화 되어있음
2. Thread-Safe
3. 계층적인 로그 설정 과 처리를 지원
4. 여러 출력을 지원 한다. (파일, 콘솔, IOStream, TCP 등)
5. 6가지 로그레벨을 지원한다. (TRACE, DEBUG, INFO, WARN, ERROR, FATAL)

### 구조
1. Logger (Category) : Log4j 의 핵심 클래스, 실제 로그 기능을 수행한다.
2. Appender : 로그 출력 위치를 지정한다.
   - ConsoleAppender : 콘솔
   - FileAppender : 파일
   - RollingFileAppender : 로그 파일의 크기가 지정한 용량을 넘어서면 다른 이름의 파일을 사용
   - DailyRollingFileAppender : 1일 단위로 로그 메시지를 파일에 출력
   - SMTPAppender : 로그 메시지를 이메일로 전송
   - NTEventLogAppender : 윈도우 이벤트 로그 시스템에 기록
3. Layout : 로그의 출력 포맷을 지정한다.
   - %d : 로그의 기록시간을 출력한다.
   - %p : 로깅의 레벨을 출력한다. 
   - %F : 로깅이 발생한 프로그램의 파일명을 출력한다.
   - %M : 로깅이 발생한 메소드의 이름을 출력한다.
   - %l : 로깅이 발생한 호출지의 정보를 출력한다.
   - %L : 로깅이 발생한 호출지의 라인수를 출력한다.
   - %t : 로깅이 발생한 Thread명을 출력한다.
   - %c : 로깅이 발생한 카테고리를 출력한다.
   - %C : 로깅이 발생한 클래스명을 출력한다.
   - %m : 로그 메시지를 출력한다.
   - %n : 개행 문자를 출력한다.
   - %% : %를 출력
   - %r : 어플리케이션이 시작 이후부터 로깅이 발생한 시점까지의 시간을 출력한다.(ms)
   - %x : 로깅이 발생한 Thread 와 관련된 **NDC(nested diagnostic context)** 를 출력한다.
   - %X : 로깅이 발생한 Thread 와 관련된 **MDC(mapped diagnostic context)** 를 출력한다.
4. Logging-Level
   1. FATAL : 가장 크리티컬한 에러
   2. ERROR : 일반적인 에러
   3. WARN : 에러는 아니지만 주의할 필요가 있을때
   4. INFO : 일반적인 정보
   5. DEBUG : 일반적인 정보를 상세히 표시할 때
   6. TRACE : 경로 추적

### 장/단점
`장점`
1. 프로그램 문제 파악이 용이
2. 빠른 디버깅
3. 쉬운 로그파악
4. 다양한 로그출력
5. 효율적인 디버깅


`단점`
1. 로그에 대한 입출력으로 인한 오버헤드 발생
2. 로깅을 위한 추가코드로 인해 전체 코드 사이즈 증가
3. 과도한 로그는 혼란을 야기할 수 있음
4. 과도한 로그는 애플리케이션 성능에 영향
5. 개발 도중 로깅코드를 추가하기 어려움


## Logback
**Logback** 은 Log4j 를 기반으로 만든 Logging 라이브러리이다.
SLF4j 를 통해 다른 로깅라이브러리와 마찬가지로 logback 으로 손쉬운 통합이 가능하다.
Log4j 와 유사한 구조를 가지고 있으며, 주요 요소들은 다음과 같다.

1. Logger : 실제 로깅을 수행한다. Level 속성을 통해 로그의 레벨 설정 가능
2. Appender : 로그 메세지가 출력될 대상을 결정
3. Encoder : Appender 에 포함되어 지정된 포맷으로 로그를 변환하는 역할

### 장점
Log4j 보다 약 10배 정도 성능이 빨라 지고, 메모리 효율도 좋아졌다.
문서화가 잘되어 있다.
설정파일 변경시 서버 재기동 없이 변경내용이 반영 된다
서버 중지 없이 I/O Failure 에 대한 복구를 지원한다.
RollingFileAppender 를 사용할 경우 자동적으로 오래된 로그를 지워주며 롤링 백업을 지원한다.

### 설정
로그백은 logback-core, logback-classic, logback-access 3개의 모듈로 구성되어 있다.
이중 logback-core 는 classic 과 access 의 공통 라이브러리 이다.

로그백은 그루비 (Groovy) 도 지원한다.
logback.groovy, logback-test.xml, logback.xml 파일이 classpath 에 존재하지 않으다면 디폴트 설정으로 동작하게 된다.
각 설정은 우선순위가 존재한다.
1. logback.groovy
2. logback-test.xml
3. logback.xml

### 로깅 레벨
log4j 와는 다르게 로깅레벨은 5단계로 구성된다

1. ERROR : 일반 에러
2. WARN : 에러는 아니지만 주의
3. INFO : 일반 정보
4. DEBUG : 일반 정보를 상세히 표시
5. TRACE : 경로 추적

## MDC 란 ?

하나의 프로그램은 여러 메소드들의 조합으로 완성된다.
요청처리를 위해 정해진 순서대로 메소드가 호출되어야 하는데, 멀티스레드 환경에서 여러 스레드에서 각각 처리하는 중 각 메소드에서 로그를 남기게되면
멀티스레드에서는 스레드들이 컨텍스트를 서로 바꿔가며 실행되기 때문에 로그 메시지가 섞이게 된다.
이런 문제를 해결하기 위해 로그 기록시 요청마다 고유의 아이디를 부여해, 로그를 기록하게되면 ID 를 이용해 그루핑할 수 있다.
이런 아이디를 **Correlation ID** 라고 한다.

**Correlation ID** 를 구현하기 위해서는 자바를 기준으로 ThreadLocal 을 사용해 해결할 수 있다.
하지만, 이를 일일히 모두 구현하기엔 불편할 수 있기 때문에 Log4j, Logback 등에서는 이런 기능을 **MDC (Mapped Diagnostic Context)** 로 제공한다.
단순히 Correlation ID 뿐 아니라 map 형태로 메타데이터를 추가할 수 있다.
주의할 점은 Correlation ID는 *다양한 요청에 대한 컨텍스트는 알 수 없다* 는 점이다.

`MDC 를 사용한 예제`
```java
class App {
    private static Logger log = LoggerFactory.getLogger(App.class);
    public static void main(String[] args) {
        log.info("Hello");
        
        MDC.put("id", "ncucu.me");
        MDC.put("username", "준영");
        
        log.info("MDC TEST");
        
        MDC.clear();
        
        log.info("== AFTER MDC CLEAR ==");
    }
}
```

스프링부트에서 JSON Format 으로 Logging 을 하기위해 아래 의존성 추가
```xml
<dependency>
   <groupId>ch.qos.logback.contrib</groupId>
   <artifactId>logback-json-classic</artifactId>
   <version>0.1.5</version>
</dependency>

<dependency>
   <groupId>ch.qos.logback.contrib</groupId>
   <artifactId>logback-jackson</artifactId>
   <version>0.1.5</version>
</dependency>
```

`로그 출력 예시`
```json
{"timestamp":"2021-01-03T11:35:36.408Z","level":"INFO","thread":"main","logger":"me.june.spring.Application","message":"Hello","context":"default"}
{"timestamp":"2021-01-03T11:35:36.452Z","level":"INFO","thread":"main","mdc":{"id":"ncucu.me","username":"준영"},"logger":"me.june.spring.Application","message":"MDC TEST","context":"default"}
{"timestamp":"2021-01-03T11:35:36.453Z","level":"INFO","thread":"main","logger":"me.june.spring.Application","message":"== AFTER MDC CLEAR ==","context":"default"}
{"timestamp":"2021-01-03T11:35:36.687Z","level":"INFO","thread":"restartedMain","logger":"me.june.spring.Application","message":"Hello","context":"default"}
{"timestamp":"2021-01-03T11:35:36.687Z","level":"INFO","thread":"restartedMain","mdc":{"id":"ncucu.me","username":"준영"},"logger":"me.june.spring.Application","message":"MDC TEST","context":"default"}
{"timestamp":"2021-01-03T11:35:36.688Z","level":"INFO","thread":"restartedMain","logger":"me.june.spring.Application","message":"== AFTER MDC CLEAR ==","context":"default"}
```

## 참고
- https://goddaehee.tistory.com/45
- https://bcho.tistory.com/1316