# 더 자바, 애플리케이션을 테스트하는 방법

## TestContainers 정보를 스프링 테스트에서 참조하기
- Spring 의 ApplicationContextIntializer 를 활용하는 방법

```java
@SpringBootTest
@ExtendWith(MockitoExtension.class)
@ActiveProfiles("test")
@Testcontainers
@Slf4j
@ContextConfiguration(initializers = StudyServiceTest.ContainerPropertyInitializer.class)
class StudyServiceTest {

	@Autowired
	Environment environment;

	@Value("${container.port}")
	int port;

	@Container
	static GenericContainer container = new GenericContainer("postgres") // imageName 은 로컬에서 찾아보고 없다면 원격에서 찾아온다.
			.withExposedPorts(5432) // port는 설정이 불가능하고, 사용가능한 포트중에서 랜덤하게 매핑한다.
			.withEnv("POSTGRES_DB", "studytest")
			;

    @BeforeAll
	static void setUp() {
    	container.start();
	}

	@AfterAll
	static void clean() {
    	container.stop();
	}


	@BeforeEach
	void beforeEach() {
		System.out.println(environment.getProperty("container.port"));
		System.out.println("port = " + port);
	}

    static class ContainerPropertyInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {

		@Override
		public void initialize(ConfigurableApplicationContext context) {
			// 가변인자로 여러개를 구성할 수 있다.
			// 단점은 key=value 의 문자열 형태로 넘겨주어야 한다
			TestPropertyValues.of("container.port=" + container.getMappedPort(5432))
				.applyTo(context.getEnvironment()); // Spring Environment 객체에 등록
		}
	}
}
```  
- @ContextConfiguration
    - 스프링이 제공하는 애노테이션으로, 스프링 테스트 컨텍스트가 사용할 설정 파일 또는 컨텍스트를 커스터마이징할 수 있는 방법을 제공한다.
- ApplicationContextInitializer
    - 스프링 ApplicationContext를 프로그래밍으로 초기화 할 때 사용할 수 있는 콜백 인터페이스로, 특정 프로파일을 활성화 하거나, 프로퍼티 소스를 추가하는 등의 작업을 할 수 있다.
- TestPropertyValues
    - 테스트용 프로퍼티 소스를 정의할 때 사용한다.
- Environment
    - 스프링 핵심 API로, 프로퍼티와 프로파일을 담당한다.

`Flow`

- TestContainers 를 이용하여 컨테이너를 생성한다.
- ApplicationContextInitializer 를 구현하여 생성된 컨테이너에서 정보를 추출하여 Environment 에 넣어준다.
- @ContextConfiguration 애노테이션을 사용하여 구현체를 등록한다.
- Environment / @Value / @ConfigurationProperties 등을 활용하여 해당 프로퍼티에 접근한다.
