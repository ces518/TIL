# Spring 5 in Action

## 8장 비동기 메시지 전송하기
- **비동기 (asynchronous)** 메시징은 애플리케이션간 응답을 대기하지 않고 간접적으로 메시지를 전송하는 방법
- 이는 애플리케이션의 결합도를 낮추고, 확장성을 높혀준다.
- 스프링이 제공하는 3가지 비동기 메시지가 있다.
1. JMS (Java Message Service)
2. RabbitMQ (AMQP, Advanced Message Queueing Protocol) 
3. Kafka (Apache Kafka)

### Blocking 과 NonBlocking, Synchronous 와 Asynchronous
- 차이를 설명할수 있나요 ?
- Blocking 과 NonBlocking 는 함수의 **제어권** 과 관련이 있다.
  - 이는 주로 I/O 를 예로 많이 든다.
  - 대표적으로는 JDBC.. 커넥션풀을 많이 주더라도 JDBC 는 Blocking 방식으로 동작하기 때문에, 동시에 처리가능한 것은 CPU 코어수를 넘지 못한다.
- Synchronous 와 Asynchronous 는 추상적인 개념이다.
  - 주로 작업 단위를 예로 많이 든다.
  - 작업이 완료되면 호출된 함수가 callback 함수를 실행해 완료 여부를 알려준다면 Asynchronous
  - 메인 스레드가 주기적으로 작업이 완료됨을 확인한다면 ? Synchronous
- 뭔가 비슷하면서도 다른 느낌적인 느낌 ? ...

> Blocking-Async 의 조합은 ???.. 띠용.. 왜쓰는거지 ?
> Node-MySQL, WebFlux-RDB 조합이 이런 느낌...

- 실무에 대입해보자면....
  - 모든건 상대적인 요소
  - 비즈니스가 중요한 플랫폼적인 성향을 가진 앱 -> 상품, 주문 등.. (WebMvc-RDB)
  - 사용자에게 노출하고, 전시적인 요소가 강하고, 트래픽을 받아내야한다. (WebFlux-NoSQL)

- https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/

### JMS

#### JMS 란?
`위키 백과`
```
자바 메시지 서비스(Java Message Service; JMS)는 자바 프로그램이 네트워크를 통해 데이터를 송수신하는 자바 API이다.

자바 메시지 서비스 API는 두 개 혹은 그 이상의 클라이언트 간 메시지 통신을 위한 자바 메시지 기반 미들웨어 API(자바 메시지 지향 미들웨어 (MOM) API)이다.
JMS는 자바 플랫폼, 엔터프라이즈 에디션에 포함되어 있으며, 자바 커뮤니티 프로세스의 **JSR 914** 로 개발된 명세서에 의해 정의된다.
자바 메시지 서비스는 자바 플랫폼, 엔터프라이즈 에디션에 기반을 둔 애플리케이션 컴포넌트들끼리 메시지를 생성, 송/수신, 읽기 기능을 제공하는 메시징 표준이다. 
분산된 애플리케이션끼리 느슨하게 연결해주고 신뢰성을 보장하며, 비동기 처리가 가능하도록 해준다.
```

> 한줄 요약 -> **JMS (Java Message Service)** 란, 자바 메시징의 표준이다.
- https://docs.oracle.com/cd/E19435-01/819-2222/concepts.html

#### JMS 설정
- JMS 를 사용하기 위해서는 JMS 클라이언트를 클래스패스에 추가해 주어야 한다.
-  
- spring-boot-starter-activemq, spring-boot-starter-artemis

