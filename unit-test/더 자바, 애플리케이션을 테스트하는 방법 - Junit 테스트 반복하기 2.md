# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 테스트 반복하기 - 2
- @ParameterizedTest 자세히 살펴보기
    - @ValueSource
        - 다양한 타입의 값들을 인자로 넣어줄 수 있다.
    - @NullSource
        - 테스트 인자로 null 을 추가해준다.
    - @EmptySource
        - 테스트 인자로 빈 문자열을 추가해준다.
    - @NullAndEmptySource
        - 테스트 인자로 null 과 빈 문자열을 추가해준다.
    - @EnumSource
        - 테스트 인자로 Enum 을 넣어줄 수 있다.
    - @CsvSource
        - 테스트 인자로 csv 형식으로 넣어줄 수 있다.

> Junit 5 에서는 기본적으로 제공하지만 만약 Junit4 를 사용중이라면 서드파티 라이브러리가 필요하다.

`다양한 Parameter Source 제공`

```java
class StudyTest {
    // Junit 5 는 기본 제공, Junit4 는 서드파티 라이브러리 필요
    @DisplayName("파라미터 테스트")
    @ParameterizedTest(name = "{displayName}, {index} {arguments}")
    @ValueSource(strings = {
        "hello",
        "world"
    })
    @NullSource
    @EmptySource
    @NullAndEmptySource
    void parameterizedTest(String value) {
        System.out.println("value = " + value);
    }
}
```

- 인자 값의 타입 변환을 지원
    - 암묵적인 타입 변환
      - [공식문서](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-argument-conversion-implicit)
    - 명시적인 타입 변환
      - SimpleArgumentConverter 를 상속받은 구현체를 제공해야 한다.
        - 하나의 인자값만 변환이 가능하다.
      - @ConvertWith

`ArgumentConverter`

```java
class StudyTest {
    
    @DisplayName("ArgumentConverter Test")
    @ParameterizedTest
    @ValueSource(ints = {10, 20, 40})
    void studyConverterTest(@ConvertWith(StudyConverter.class) Study study) {
        System.out.println("study = " + study);
    }

    // ArgumentConverter 는 하나의 인자값만 변환 가능
    static class StudyConverter extends SimpleArgumentConverter {

        @Override
        protected Object convert(Object target, Class<?> type) throws ArgumentConversionException {
            assertEquals(Study.class, type, "Can only convert to Study");
            return new Study(Integer.parseInt(target.toString()));
        }
    }
}
```

- 인자 값 조합
    - ArgumentsAccessor
    - 커스텀 Accessor
        - ArgumentsAggregator 인터페이스를 구현해야 한다.
        - @AggregateWith

`ArgumentAggregator`

```java
class StudyTest {
    @ParameterizedTest
    @CsvSource({"10, '자바 스터디'", "20, '스프링'"})
    void studyCsvSourceTest(Integer limit, String name) {
        System.out.println("new Study(limit, name) = " + new Study(limit, name));
    }

    @ParameterizedTest
    @CsvSource({"10, '자바 스터디'", "20, '스프링'"})
    void studyCsvSourceAggregateTest(@AggregateWith(StudyAggregator.class) Study study) {
        System.out.println("Study = " + study);
    }

    // 반드시 static inner 혹은 public class 여야 한다.
    static class StudyAggregator implements ArgumentsAggregator {

        @Override
        public Object aggregateArguments(ArgumentsAccessor argumentsAccessor, ParameterContext parameterContext) throws ArgumentsAggregationException {
            return new Study(argumentsAccessor.getInteger(0), argumentsAccessor.getString(1));
        }
    }
}
```