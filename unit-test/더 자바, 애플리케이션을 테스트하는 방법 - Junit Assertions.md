# 더 자바, 애플리케이션을 테스트하는 방법

## Junit Assertions
- 테스트에서 검증하고자 하는 것을 확인 하는 기능
- [공식 문서](https://junit.org/junit5/docs/current/user-guide/#writing-tests-assertions)
- AssertJ, Hemcrest, Truth 등의 라이브러리를 사용할 수도 있다.

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

	@Test
	@DisplayName("스터디 생성")
	void create() {
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
}
```