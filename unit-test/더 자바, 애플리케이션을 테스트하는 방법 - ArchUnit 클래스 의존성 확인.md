# 더 자바, 애플리케이션을 테스트하는 방법

## ArchUnit JUnit 클래스 의존성 확인

```java
@AnalyzeClasses(packagesOf = TestApplication.class)
class ArchTests {
    @ArchTest // Controller 클래스는 Service 와 Repository 를 참조할 수 있다.
    ArchRule controllerClassRule = classes().that().haveSimpleNameEndingWith("Controller")
        .should().accessClassesThat().haveSimpleNameEndingWith("Service")
        .orShould().accessClassesThat().haveSimpleNameEndingWith("Repository");

    @ArchTest // Repository 는 Service 를 참조할 수 없다.
    ArchRule repositoryClassRule = noClasses().that().haveSimpleNameEndingWith("Repository")
        .should().accessClassesThat().haveSimpleNameEndingWith("Service");

    @ArchTest // Study 로 시작하는 클래스 중 Enum , @Entity 애노테이션을 사용하지 않은 클래스는 study 패키지 내에 존재해야 한다.
    ArchRule studyClassesRule = classes().that().haveSimpleNameStartingWith("Study")
        .and().areNotEnums()
        .and().areNotAnnotatedWith(Entity.class)
        .should().resideInAnyPackage("..study..");
}
```

- 상속 관계 테스트, 특정 클래스는 특정 패키지 내에서만 액세스 가능, 특정 아키텍쳐에 맞게 구성되어 있는지 (layered, onion) 등 다앙한 기능이 존재한다.
- PlantUML 로 그린 Diagram 기반으로 확인도 가능하다.