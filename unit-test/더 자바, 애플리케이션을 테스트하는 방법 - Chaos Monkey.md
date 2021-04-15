# 더 자바, 애플리케이션을 테스트하는 방법

## Chaos Monkey
- 카오스 엔지니어링 툴
    - 프로덕션 환경, 특히 분산 시스템 환경에서 불확실성을 파악하고 해결방안을 모색하는 사용 하는 툴
- 운영 환경 불확실성의 예
    - 네트워크 지연
    - 서버 장애
    - 디스크 오작동
    - 메모리 누수
    - ... 등
    
> 운영 환경에서 경험할 수 있는 다양한 이슈들에 대한 테스팅

- 카오스 멍키 스프링 부트
    - 스프링 부트 애플리케이션에 카오스 멍키를 손쉽게 적용 가능
    - 스프링 부트 앱을 망가트릴 수 있는 툴

- 주요 개념
    - 애노테이션이 적용되어 있는 퍼블릭 메소드를 사용할때 다양한 공격이 가능함
    - 응답지연, 예외, 애플리케이션 종료, 메모리 누수 등..


| 공격 대상 (Watcher) | 공격 유형 (Assaults) |
| --- | --- |
| @RestController <br/> @Controller <br/> @Service <br/> @Repository <br/> @Component | 응답 지연 (Latency Assault) <br/> 예외 발생 (Exception Assault) <br/> 애플리케이션 종료 (AppKiller Assault) <br/> 메모리 누수 (Memory Assault) |







