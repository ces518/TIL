# 실전! Querydsl - 라이브러리 살펴보기
> http://querydsl.com/

#### 라이브러리 살펴보기
- querydsl-apt
    - code generate 용도로 사용하는 라이브러리
- querydsl-jpa
    - 실제 애플리케이션 작성시 필요한 라이브러리
- spring data jpa
    - hibernate
- hikari cp
    - 데이터베이스 커넥션풀
- spring-boot-starter-web
    - spring core
    - embbed tomcat
- spring-boot-starter-logger
    - 로깅 프레임워크
    - slf4j 사용 (로깅 퍼사드)
    - logback (실제 구현체)


#### 테스트 라이브러리
- spring-boot-starter
    - junit
        - 스프링 부터 2.2부터 junit5를 사용한다.
        - 과거 버전은 vintage 사용
    - mockito
        - 목 라이브러리
    - assertj
        - 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
    - spring-test
        - 스프링 통합 테스트 지원

`핵심 라이브러리`
- Spring MVC
- JPA, Hibernate
- Spring data JPA
- Querydsl

`기타 라이브러리`
- H2 Database
- HikariCP
- SLF4J, LogBack
- 테스트
