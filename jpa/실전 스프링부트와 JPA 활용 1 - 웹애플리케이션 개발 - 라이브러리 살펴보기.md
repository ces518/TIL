# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 라이브러리 살펴보기

#### 라이브러리 살펴보기
- 현재 의존성 살펴보기 (gradle)
    - 프로젝트 루트 디렉터리로 이동
    - ./gradlew dependencies 

`주요 라이브러리`
- Spring Boot Starter Web
    - embed-tomcat (내장톰켓)
    - spring webmvc (Spring MVC)
- Spring Boot Starter Thymeleaf (템플릿 엔진)
- Spring Boot Starter data JPA (JPA)
    - spring boot starter aop
    - spring boot starter jdbc
        - HikariCP (Spiring2.x 부터 기본 커넥션풀)
        - spring jdbc (트랜잭션, jdbcTemplate 등..)
    - hibernate
    - spring data jpa
- Spring Boot Starter Test
    - JUnit
    - spring Test
    - mockito
    - assertJ

- Spring Boot는 기본적으로 로그백을 사용한다.
    - slf4j (로깅 퍼사드) - logback(구현체)
