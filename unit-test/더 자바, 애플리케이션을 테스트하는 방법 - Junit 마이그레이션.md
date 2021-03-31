# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 마이그레이션
- Junit5 가 지원하는 Junit4 마이그레이션 지원 기능
- 기본적으로 Spring Boot 로 프로젝트를 생성하면 junit-vintage-engine 이 의존성에서 제외되어 있다.
- junit-vintage-engine 이 존재하면 Junit 3과 4로 작성된 테스트를 실행할 수 있다.
- Junit 5 가 가지고 있는 Junit Platform 을 이용해서 실행한다.

## 제약 사항
- @Rule (Junit4) 은 기본적으로 지원하지 않는다.
    - junit-jupiter-migrationsupport 모듈을 추가해야 한다.
- @EnableRuleMigrationSupport 를 사용하면 아래 세가지 Rule 을 지원한다.
    - ExternalResource
    - Verifier
    - ExpectedException
> 하지만 완벽하게 지원한다고는 할 수 없다.

| Junit4 | Junit5 |
| --- | --- |
| @Category(Class) | @Tag(String) |
| @RunWith, @Rue, @ClassRule | @ExtendWith, @RegisterExtension |
| @Ignore | @Disabled |
| @Before, @After, @BeforeClass, @AfterClass | @BeforeEach, @AfterEach, @BeforeAll, @AfterAll |

