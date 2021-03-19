# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 테스트 이름 표기하기
- Junit 테스트 결과에서 표시하는 테스트 이름의 기본 전략은 **메소드 명** 이다.
- 그 외에 좀 더 명확하게 표기하는 방법을 제공한다.

## @DisplayNameGeneration
- @DisplayNameGeneration 애노테이션을 사용해서 테스트 결과에 출력되는 테스트 명의 생성 전략을 변경할 수 있다.
- @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class) 로 지정하면, 언더스코어를 공백으로 치환해주게 된다.

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

	@Test
	void create() {
		Study study = new Study();
		assertNotNull(study);
	}
}
```

## @DisplayName
- 메소드 명만으로는 테스트에 대한 디스크립션을 충분히 하지 못하는 경우가 많다
- 언더스코어로 구분을 하게 되면 불필요하게 메소드명이 길어지는 경우가 생긴다.
- 이런 경우 @DisplayName 애노테이션을 사용해서 디스크립션을 달 수 있다.
- 테스트 결과에서도에 @DisplayName 으로 지정한 디스크립션이 결과에 출력되게 된다.

```java
class StudyTest {

	@Test
	@DisplayName("스터디 생성")
	void create() {
		Study study = new Study();
		assertNotNull(study);
	}
}
```