# 실전! Querydsl - SpringBoot 설정 JPA,DB

#### SpringBoot 설정 JPA, DB
`application.yml`
```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/querydsl
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
#        show_sql: true
        format_sql: true


logging.level:
  org.hibernate.SQL: debug
```

- Connection 및 logging 설정

`build.gradle`
```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // querydsl 라이브러리 추가
    implementation 'com.querydsl:querydsl-jpa'
    implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.7'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}
```

- p6spy 라이브러리추가
- Query Parameter Logging 기능

> 쿼리 로깅같은 경우에는 성능 테스트 이후 사용할것. 성능이 중요한 시스템의 경우에는 고려를 해 보아야 한다.
