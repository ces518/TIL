# 실전! 스프링 데이터 JPA - 라이브러리 살펴보기

#### gradle 의존관계 살펴보기
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}
```

> Spring 2.2.x 부터 junit5 를 기본 테스트 의존성으로 가져 간다.
- org.junit.vintage는 junit4와 호환되는 라이브러리이다. 따라서 제외시켜버린것

#### 핵심 라이브러리
- Spring MVC
- Spring ORM
- JPA, Hibernate
- Spring data JPA

#### 기타 라이브러리
- H2
- HikariCP
- SLF4J(로깅퍼사드) & Logback(구현체)
- 테스트

> H2 라이브러리 버전과 H2 클라이언트 버전을 맞추어 주어야한다.
