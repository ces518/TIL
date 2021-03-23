# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 테스트 반복하기 - 1
- 테스트를 반복적으로 실행하는 방법
- @RepeatedTest
    - 테스트를 일정한 횟수만큼 반복하는 방법
    - 랜덤한 값을 생성해서 테스트를 하는 등 반복적인 검증이 필요한 경우 유용하다.
- @ParameterizedTest
    - 지정한 파라미터 목록을 이용해 테스트를 반복하는 방법
    - 특정한 값을 지정해서 해당 값을 기반으로 검증이 필요한 경우 유용하다.

```java
class StudyTest {
    
    @DisplayName("반복 테스트")
    @RepeatedTest(value = 10, name = "{displayName}, {currentRepetition}/{totalRepetitions}")
    void repeatTest(RepetitionInfo repetitionInfo) {
        // 현재 몇번째 반복중인지
        // 총 몇번을 반복해야 하는지 정보를 알 수 있음
        System.out.println("repeatTest" + repetitionInfo.getCurrentRepetition() + "/" + repetitionInfo.getTotalRepetitions());
    }

    // Junit 5 는 기본 제공, Junit4 는 서드파티 라이브러리 필요
    @DisplayName("파라미터 테스트")
    @ParameterizedTest(name = "{displayName}, {index} {arguments}")
    @ValueSource(strings = {
        "hello",
        "world"
    })
    void parameterizedTest(String value) {
        System.out.println("value = " + value);
    }
}
```