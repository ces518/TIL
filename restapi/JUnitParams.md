# REST API - 테스트코드 Refactoring
- 테스트코드의 중복을 제거하는 방법은 여러 가지가 있지만, JunitParams 를 사용할것

#### JUnitParams
- https://github.com/Pragmatists/JUnitParams
- JUnit Method는 Parameter를 사용할 수 없지만, Parameter를 사용할수 있게 해주며 Parameter를 활용하여 중복을 제거할 수 있다.
- 의존성 추가
```xml
<dependency>
    <groupId>pl.pragmatists</groupId>
    <artifactId>JUnitParams</artifactId>
    <version>1.1.1</version>
    <scope>test</scope>
</dependency>
```

- 테스트 코드
    - @Parameters에 해당하는 값들이 들어간채로 호출하여 여러번 테스트도 가능하다.
    - 하지만 이 방법은 Type-Safe 하지않다. 이에 대한 대안도 존재한다.
```java
@Test
@Parameters({
        "0, 0, true",
        "100, 0, false",
        "0, 100, false"
})
public void testFree (int basePrice, int maxPrice, boolean isFree) {
    // given
    Event event = Event.builder()
            .basePrice(basePrice)
            .maxPrice(maxPrice)
            .build();

    // when
    event.update();

    // then
    assertThat(event.isFree()).isEqualTo(isFree);

    // given
    event = Event.builder()
            .basePrice(basePrice)
            .maxPrice(maxPrice)
            .build();

    // when
    event.update();

    // then
    assertThat(event.isFree()).isEqualTo(isFree);
}
```

- Type-Safe한 방법
    - Parameters로 제공할 Object[]를 return 하는 Method를 정의한다.
    - Method Convention 에 따라 method 명을 생략할 수 있다.
    - parametersFor[테스트할 메서드명] 의 컨벤션 에 따라 method = "파라메터 제공 메서드" 를 생략 할 수 있다.
```java
private Object[] parametersForTestFree () {
    return new Object[] {
            new Object[] {0, 0, true},
            new Object[] {100, 0, false},
            new Object[] {100, 200, false}
    };
}

@Parameters(method = "parametersForTestFree")
public void testFree (int basePrice, int maxPrice, boolean isFree) {
    // given
    Event event = Event.builder()
            .basePrice(basePrice)
            .maxPrice(maxPrice)
            .build();

    // when
    event.update();

    // then
    assertThat(event.isFree()).isEqualTo(isFree);

    // given
    event = Event.builder()
            .basePrice(basePrice)
            .maxPrice(maxPrice)
            .build();

    // when
    event.update();

    // then
    assertThat(event.isFree()).isEqualTo(isFree);
}
```
