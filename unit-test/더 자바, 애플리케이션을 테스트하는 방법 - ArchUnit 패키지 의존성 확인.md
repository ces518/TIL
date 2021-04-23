# 더 자바, 애플리케이션을 테스트하는 방법

## ArchUnit 패키지 의존성 확인

```java
class ArchTests {

    @Test
    void packageDependencyTests() {
        // 클래스 목록 읽어오기
        JavaClasses classes = new ClassFileImporter().importPackages("me.june.test");

        /**
         * ..domain.. 패키지에 있는 클래스는 ..study.., ..member.., ..domain에서 참조 가능.
         * ..member.. 패키지에 있는 클래스는 ..study..와 ..member..에서만 참조 가능.
         * (반대로) ..domain.. 패키지는 ..member.. 패키지를 참조하지 못한다.
         * ..study.. 패키지에 있는 클래스는 ..study.. 에서만 참조 가능.
         * 순환 참조 없어야 한다.
         */

        // domain 패키지 는 study, member, domain 패키지에서만 참조할 수 있다.
        ArchRule domainPackageRule = classes().that().resideInAPackage("..domain..")
            .should().onlyBeAccessed().byClassesThat().resideInAnyPackage("..study..", "..member..", "..domain..");

        domainPackageRule.check(classes);

        // domain 패키지에 있는 어떤 클래스도 member 클래스내에 있는 클래스에 접근할 수 없다.
        ArchRule memberPackageRule = noClasses().that().resideInAPackage("..domain..")
            .should().accessClassesThat().resideInAPackage("..member..");

        memberPackageRule.check(classes);

        // study 패키지 외에서는 study 패키지에 접근할 수 없다.
        ArchRule studyPackageRule = noClasses().that().resideOutsideOfPackage("..study..")
            .should().accessClassesThat().resideInAPackage("..study..");

        studyPackageRule.check(classes);

        // 순환참조 체크
        ArchRule freeOfCycles = slices().matching("..test.(*)..")
            .should().beFreeOfCycles();

        freeOfCycles.check(classes);
    }
}
```