# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 커스텀 태그
- Junit 이 제공하는 애노테이션 들은 메타 애노테이션이다.
- 우리가 커스텀한 애노테이션들을 만들어 사용할 수 있다.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Test
@Tag("fast")
public @interface FastTest {
}
```

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

	@FastTest
    public void fasttest() {
		
    }
}
```