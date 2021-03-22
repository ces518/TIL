# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 태깅과 필터링
- 테스트 그룹을 만들고 원하는 테스트 그룹만 실행할 수 있는 기능
- 빌드 툴에서는 기본적으로 모든 테스트를 실행한다.

`Maven 설정`

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <groups>fast | slow</groups>
    </configuration>
</plugin>
```

```
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

    @Tag("fast")
    @Test
    @Disabled
    void disabled() {

    }
}
```