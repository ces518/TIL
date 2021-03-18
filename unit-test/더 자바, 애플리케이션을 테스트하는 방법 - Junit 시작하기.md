# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 시작하기
- Spring Boot 2.2.x 부터는 starter-test 가 제공하는 Junit 버전이 5로 올라갔다.
- Junit 4 까지는, 테스트 클래스와 테스트 메소드가 **public** 이여야 실행이 가능했다.
- 하지만 Junit 5 부터는 public 여야 할 제약사항이 사라졌다.

## Junit 에서 제공하는 전/후처리 애노테이션

```java
class StudyTest {

    @Test
    void create() {
        Study study = new Study();
        assertNotNull(study);
    }

    /**
        @BeforeAll
        모든 테스트 클래스가 실행되기 전에 딱 1회만 실행된다.
        static 메소드로 정의 되어야 한다.
        private 접근 제어자는 사용할 수 없다.
     */
    @BeforeAll
    static void beforeAll() {
        System.out.println("beforeAll");
    }

    /**
        @AfterAll
        모든 테스트 클래스가 종료된 후 딱 1회만 실행된다.
        static 메소드로 정의 되어야 한다.
        private 접근 제어자는 사용할 수 없다.
    */
    @AfterAll
    static void afterAll() {
        System.out.println("afterAll");
    }

    /**
        @BeforeEach
        각 테스트가 실행되기전 실행된다.
        static 메소드일 필요가 없다.
     */
    @BeforeEach
    void beforeEach() {
        System.out.println("beforeEach");
    }

    /**
         @AfterEach
         각 테스트가 실행된 후 실행된다.
         static 메소드일 필요가 없다.
     */
    @AfterEach
    void afterEach() {
        System.out.println("afterEach");
    }
}
```

`실행 결과`

```text
beforeAll
beforeEach
afterEach
afterAll
```

## 테스트 코드를 비활성화 시키는 애노테이션

```java
class StudyTests {

    @Test
    @Disabled
    void disabled() {

    }
}
```