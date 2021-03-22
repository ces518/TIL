# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 조건에 따라 테스트 실행하기
- 특정 OS, Java Version, 환경 변수에 따라서 테스트가 실행되게 할수 있는 기능
- org.junit.jupiter.api.Assumptions.* 패키지에 존재하는 메소드들을 사용할 수 있으며, @Enabled 와 같은 애노테이션도 제공한다.  

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

	@Test
	@DisplayName("스터디 생성")
	@EnabledOnOs(OS.MAC) // Annotation 으로 제공
	void create() {
		// 특정 환경변수에 따라 테스트를 실행
		assumeTrue("LOCAL".equalsIgnoreCase(System.getProperty("TEST_ENV")));

		// 특정 환경에 따라 실행
		assumingThat("LOCAL".equalsIgnoreCase(System.getProperty("TEST_ENV")), () -> {
			Study study = new Study(100);
		});
	}
}
```