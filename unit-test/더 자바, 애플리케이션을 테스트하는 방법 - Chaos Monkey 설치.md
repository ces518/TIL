# 더 자바, 애플리케이션을 테스트하는 방법

## Chaos Monkey 설치

```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>chaos-monkey-spring-boot</artifactId>
    <version>2.1.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

> 의존성 추가후 반드시 chaos-monkey profile 로 실행을 해야 카오스 멍키가 활성화 된다.

