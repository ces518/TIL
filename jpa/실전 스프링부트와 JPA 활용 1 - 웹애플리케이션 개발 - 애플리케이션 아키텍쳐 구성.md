# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 애플리케이션 아키텍쳐 구성

#### 애플리케이션 아키텍쳐
- 일반적인 계층형 구조를 사용한다

`계층 구조`
- Controller > Service > Repository > DB
- Domain

- controller, web: 웹계층
- service: 비즈니스 로직, 트랜잭션 처리
- repository: JPA를 직접사용하는 계층, 엔티티 매니저 사용
- domain: 엔티티가 모여있는 계층, 모든 계층에서 사용한다.

> 서비스, 리포지토리 계층을 개발하고, 테스트 케이스를 작성해서 검증, 마지막에 웹 계층을 적용한다.

> 웹 계층에서 바로 repository를 사용할수 있도록 구현한다. 방향은 단방향으로 구현을 한다.

`패키지 구조`
- jpabook.jpashop
    - domain
    - exception
    - repository
    - service
    - web
