# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 테스트 순서
- 실행할 테스트 메소드는 특정한 순서에 실행되지만 순서가 매번 같다고 보장하지 않는다.
- 특정 순서대로 테스트를 실행하고 싶다면, Junit 이 지정한 순서에 의존해서는 안된다.
- **@TestMethodOrder** 를 이용하면 순서를 지정할 수 있다.
    - MethodOrderer 구현체를 지정한다.
    - 기본구현체
        - Alphanumeric
        - OrderAnnotation
        - Random

`@OrderAnnotation`

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class StudyTest {

    @FastTest
    @DisplayName("스터디 생성")
    @EnabledOnOs(OS.MAC)
        // Annotation 으로 제공
    void create() {
        // 특정 환경변수에 따라 테스트를 실행
        assumeTrue("LOCAL".equalsIgnoreCase(System.getProperty("TEST_ENV")));

        // 특정 환경에 따라 실행
        assumingThat("LOCAL".equalsIgnoreCase(System.getProperty("TEST_ENV")), () -> {
            Study study = new Study(100);
        });

        Study study = new Study(1);
        assertNotNull(study);

        // 람다식도 지원
        assertEquals(Study.Status.DRAFT, study.getStatus(), "스터디를 처음 만들면 상태값이 DRAFT 여야 한다.");
        assertEquals(Study.Status.DRAFT, study.getStatus(), () -> "스터디를 처음 만들면 상태값이 DRAFT 여야 한다.");
        assertTrue(study.getLimit() > 0, "스터디 참석 인원은 0 보다 커야합니다.");

        // 각 테스트 들을 한번에 검증
        assertAll(
            () -> assertEquals(Study.Status.DRAFT, study.getStatus(), "스터디를 처음 만들면 상태값이 DRAFT 여야 한다."),
            () -> assertEquals(Study.Status.DRAFT, study.getStatus(), () -> "스터디를 처음 만들면 상태값이 DRAFT 여야 한다."),
            () -> assertTrue(study.getLimit() > 0, "스터디 참석 인원은 0 보다 커야합니다.")
        );

        // 예외 검증
        IllegalArgumentException ex = assertThrows(IllegalArgumentException.class, () -> new Study(-10));
        assertEquals(ex.getMessage(), "limit 은 0보다 커야합니다.");

        // 실행 시간내에 완료되어야 하는지 검증
        assertTimeout(Duration.ofSeconds(10), () -> new Study(10));

        // 지정된 시간이 끝나면 테스트 실패로 간주하고 그냥 실패시키고 싶을경우
        // 지정한 시간이 10초가 지나면 테스트는 실패할것이기 때문에 종료시킨다.
        // 람다 코드블록 내에 존재하는 코드는 별도의 스레드에서 동작한다.
        // 따라서 ThreadLocal 기반 기능 (Security, Transactional) 등이 예상과 다르게 동작할 수 있다.
        assertTimeoutPreemptively(Duration.ofSeconds(10), () -> new Study(10));
    }

    @Order(1)
    @SlowTest
    @Disabled
    void disabled() {

    }

    @Order(2)
    @DisplayName("반복 테스트")
    @RepeatedTest(value = 10, name = "{displayName}, {currentRepetition}/{totalRepetitions}")
    void repeatTest(RepetitionInfo repetitionInfo) {
        // 현재 몇번째 반복중인지
        // 총 몇번을 반복해야 하는지 정보를 알 수 있음
        System.out.println("repeatTest" + repetitionInfo.getCurrentRepetition() + "/" + repetitionInfo.getTotalRepetitions());
    }
}
```

> Spring 에서 제공하는 @Order 애노테이션과 혼동해서는 안된다.\
> 순서의 값이 낮을수록 우선순위가 높다.