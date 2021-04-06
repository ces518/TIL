# 더 자바, 애플리케이션을 테스트하는 방법

## TestContainers 기능 살펴보기

```java
@SpringBootTest
@ExtendWith(MockitoExtension.class)
@ActiveProfiles("test")
@Testcontainers
@Slf4j
class StudyServiceTest {
	
	@Container
	static GenericContainer container = new GenericContainer("postgres") // imageName 은 로컬에서 찾아보고 없다면 원격에서 찾아온다.
			.withExposedPorts(5432) // port는 설정이 불가능하고, 사용가능한 포트중에서 랜덤하게 매핑한다.
			.withEnv("POSTGRES_DB", "studytest")
			.waitingFor(Wait.forListeningPort()) // 컨테이너가 사용 가능한지 대기했다가 사용하는 옵션
			.waitingFor(Wait.forHttp("/hello")) // 컨테이너가 사용 가능한지 대기했다가 사용하는 옵션
			;

	@BeforeAll
	static void setUp() {
		Slf4jLogConsumer logConsumer = new Slf4jLogConsumer(log);
		container.followOutput(logConsumer); // 컨테이너 내부의 로그를 스트리밍 한다.
		container.getMappedPort(5432); // 컨테이너 포트와 매핑된 로컬 포트 확인
		container.getLogs(); // 모든 로그들 출력
		container.start();
	}
}
```
- GenericContainer
    - 인자로 image name 을 받는다.
    - 먼저 로컬에서 해당 이미지를 찾아보고, 없다면 public 한 원격 저장소에서 찾는다.
- withExposedPorts
    - 컨테이너의 특정 포트와 로컬 포트를 매핑시킨다.
    - 로컬 포트는 지정이 불가능한데, 이는 의도된 것이다.
    - 사용가능한 포트 내에서 랜덤하게 지정된다.
- withEnv
    - 컨테이너 의 환경변수 지정
    - Key Value 형태로 구성된다.
- withCommand
    - 실행할 명령어
- waitingFor
    - 사용 준비 확인
- followOutput
    - 컨테이너 내부 로그 스트리밍
- getLogs
    - 전체 로그 출력