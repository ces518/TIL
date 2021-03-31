# 더 자바, 애플리케이션을 테스트하는 방법

## Mockito 시작하기
- Spring boot 2.2+ 프로젝트 생성시 의존성을 포함하고 있다.

`별도로 의존성 추가시`
```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>3.1.0</version>
    <scope>test</scope>
</dependency>


<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>3.1.0</version>
    <scope>test</scope>
</dependency>
```

- Mock 을 생성하는 방법 / 동작을 관리하는 방법 / 행동을 검증하는 방법 세가지만 알고 있다면 쉽게 테스트 작성이 가능

- 레퍼런스
    - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html