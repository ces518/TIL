# 더 자바, 애플리케이션을 테스트하는 방법

## ArchUnit JUnit 연동

```java
@AnalyzeClasses(packagesOf = TestApplication.class)
class ArchTests {

    /**
     * ArchTest 애노테이션을 사용하면, @DisplayName 을 지정할 수 없다는 점이 단점
     *
     * ArchUnit 은 Junit Engine 을 확장해서, ArchUnit JUnit Module 을 만들었다 (순수한 Jupiter Engine 을 사용하는 것이 아님)
     * -> 일반적으로 ExtendWith (Extension 확장) Register, RegisterExtension 을 활용한 Extension 확장 방식을 사용
     */

    @ArchTest
    ArchRule domainPackageRule = classes().that().resideInAPackage("..domain..")
        .should().onlyBeAccessed().byClassesThat().resideInAnyPackage("..study..", "..member..", "..domain..");

    @ArchTest
    ArchRule memberPackageRule = noClasses().that().resideInAPackage("..domain..")
        .should().accessClassesThat().resideInAPackage("..member..");

    @ArchTest
    ArchRule studyPackageRule = noClasses().that().resideOutsideOfPackage("..study..")
        .should().accessClassesThat().resideInAPackage("..study..");

    @ArchTest
    ArchRule freeOfCycles = slices().matching("..test.(*)..")
        .should().beFreeOfCycles();
}
```
- @AnalyzeClasses
    - 해당 클래스를 읽어 들여 확인할 패키지를 지정한다.
    - packages (문자열 기반), packageOf (Class 기반, Type Safe) property 사용
- @ArchTest
    - 확인할 규칙을 정의한다.
    - 이전에 살펴본 ArchRule 타입에 지정할 수 있다.
    
> 장점은 코드가 간결해지지만, 단점으로는 @DisplayName 을 지정할 수 없다. \n
> ArchUnit 은 Junit Engine 을 확장해서, ArchUnit JUnit Module 을 만들었다 (순수한 Jupiter Engine 을 사용하는 것이 아님)
    