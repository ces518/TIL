# 더 자바, 애플리케이션을 테스트하는 방법

## ArchUnit 설치

`의존성 추가`

```xml
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5-engine</artifactId>
    <version>0.12.0</version>
    <scope>test</scope>
</dependency>

```


1. 특정 패키지에 해당하는 클래스들을 (바이트 코드를 이용해) 읽어 들인다.
2. 확인할 규칙을 정의한다.
3. 읽어 들인 클래스들이 그 규칙을 잘 따르는지 확인한다.

```java
@Test
public void Services_should_only_be_accessed_by_Controllers() {
    JavaClasses importedClasses = new ClassFileImporter().importPackages("com.mycompany.myapp");

    ArchRule myRule = classes()
        .that().resideInAPackage("..service..")
        .should().onlyBeAccessed().byAnyPackage("..controller..", "..service.."); 
    // service 라는 패키지에 존재하는 것은 controller / service 에서만 참조해야 한다.

    myRule.check(importedClasses);
}

```

Junit 확장 팩을 추가로 제공한다.
- @AnalyzeClasses : 클래스를 읽어 들여 확인한 패키지 지정
- @ArchTest : 확인할 규칙 정의