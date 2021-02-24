# 2021 카프카 밋업
1. 카프카, 컨플루언트 플랫폼 새 기능 및 로드맵, Oracle CDC Source Connector 소개
2. 스프링 카프카로 프로듀서 / 컨슈머 개발하기

## 카프카, 컨플루언트 플랫폼 새 기능 및 로드맵, Oracle CDC Source Connector 소개

### Confluent 소개
- 카프카 2010년 링크드인에서 오픈소스로 개발
- 2014 년에 카프카 최초 3인이 창업

### Confluent Plaftform 6.1
- Apache Kafka 2.7 기반
- 2021년 2월 9일 GA
- 생산성 극대화
    - 다양한 언어
    - Prebuilt 에코 시스템
    - 이벤트 스트리밍 DB
        - ksqlDB
        - 일종의 이벤트 스트리밍을 분석하는 엔진
        - 실제 데이터베이스는 아니다.
- 대규모 운영
    - GUI 기반 관리 및 모니터링
        - Control Center
    - 유연한 DevOps
        - Operator Ansible
    - 동적 성능 및 유연성
        - 셀프 밸런스 클러스터링
- 운영 환경시 필수기능
    - 보안
- CP 6.1 은 카프카가 원활하게 실행되도록 높운 가용성 제공
- 카프카 ISR 에 옵저버가 추가됨..
    - 옵저버라고 해서 옵저만 하는것이 아닌 레플리카도 수행한다.
    - ISR 중 하나가 죽었을 경우 바로 ISR 로 승격해서 사용할 수 있게 사용..
- ksqlDB Query 변경점
    - ALTER 가 추가 되었다.
        - 기존에는 스트림이 변경되면 drop 한뒤 새로 만들어야 했다
        - 하지만 가용성을 유지하면서 ALTER 가 가능해짐.
    - Multiple Keys Pull Queries 기능 추가
        - Eq 쿼리 뿐이 아닌 In 쿼리가 가능해짐..


### Apache Kafka 2.7 주요 기능
- KIP 497
    - Zookeepr 를 제거하고 KIP 500 을 완료하는 과정에서 중요하는 부분
    - AlterISR API 기능 추가
    - ISR 의상태를 API 로 제어
- KIP 595
    - Core Consensus (핵심 합의) 프로토콜 을 포함하는 Raft 아키텍쳐를 위한 모듈 추가
    - KIP 500 컨트롤러중 리더를 선출하는 핵심 알고리즘
    - Raft 알고리즘
- KIP 500
    - Metadata Quorum (주키퍼가 하던 기능, 셀프 매니지로 하겠따…)
    - 여기도 Leader Follower Observer 구성…
    - 주키퍼가 하던 기능을 없애고 셀프로 수행하겠따..
    - 새롭게 추가된 3개의 Controller 노드가 3개의 ZooKeeper 노드들 대체한다.
        - 주키퍼 역할을 대신할 브로커를 두고, 해당 브로커가 Controller 역할을 수행한다.
- OGG (Oracle Golden Gate) Oracle CDC 
    - 보통 3초 정도 갭이 있다.
- 카프카 커넥터는 OGG 없이 사용 가능함 ~



## 스프링 카프카로 프로듀서 / 컨슈머 개발하기
- Kafka API 와 Spring for ApacheKafka 차이
- Spring for Apache Kafka 설정
- 프로듀서 컨슈머 설정
- Spring for Apache Kafka Test (테스트를 위한 내장 카프카를 사용할 수 있음)


### Kafka API 와 Spring for ApacheKafka 차이
- Kafka API
    - 직접 Row Level 로 생성 및 사용을 해야한다.
    - 프로듀서의 경우 대부분에 문제가 없다.
    - 컨슈머의 경우 이런 컨슈머 관련 코드가 어디로 가야할지 헤메는 경우가 많다.
- Spring for Apache Kafka
    - 프로듀서의 역할을 KafkaTemplate 로 추상화 되어 있다.
    - RoutingKafkaTemplate
        - 여러 가지 객체 타입을 하나의 템플릿으로 전송 가능
    - ReplyingKafkaTemplate
        - 응답 구조를 만들때 사용할 수 있음 
    - @KafkaListener
        - 컨슈머의 역할
- Replying Template
    - @SendTo 애노테이션을 사용
    - 카프카 요청-응답을 동기 구조로 사용할 수 있음 !!
- @EmbeddedKafka
    - 특정 브로커로 임베디드 카프카를 띄운다.
    - 카프카 설치 없이 1회성 테스트를 할수 있음 ..













