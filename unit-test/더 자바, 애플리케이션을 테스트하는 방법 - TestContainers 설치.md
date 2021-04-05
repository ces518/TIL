# 더 자바, 애플리케이션을 테스트하는 방법

## TestContainers 설치
- https://www.testcontainers.org

`TestContainers 의존성 추가`

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.15.2</version>
    <scope>test</scope>
</dependency>
```

TestContainers 는 다양한 모듈 들을 제공한다.

`Postgresql 모듈 의존성 추가`

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.15.2</version>
    <scope>test</scope>
</dependency>
```

`application-test.properties`

```properties
# tc 를 붙인 후엔 호스트 명과 포트는 필요 없다.
# spring.datasource.url=jdbc:postgresql://localhost:5432/studytest
spring.datasource.url=jdbc:tc:postgresql:///studytest
spring.datasource.username=studytest
spring.datasource.password=studytest
# TC 전용 드라이버를 사용해야 한다.
spring.datasource.driver-class-name=org.testcontainers.jdbc.ContainerDatabaseDriver
spring.jpa.hibernate.ddl-auto=create-drop
```
- TestContainer 용 jdbcUrl 을 사용해야하며, 호스트 명과 포트번호는 필요가 없다.
- 또한 TC 전용 드라이버를 사용해야 커넥션이 성립된다.

```java
@SpringBootTest
@ExtendWith(MockitoExtension.class)
@ActiveProfiles("test")
@Testcontainers
class StudyTest {
	// 필드 일 경우 모든 테스트 마다 컨테이너를 새로 띄운다.
	// 스태틱 필드일 경우 해당 테스트 클래스 전역에서 공유한다.
	@Container
	static PostgreSQLContainer container = new PostgreSQLContainer()
			.withDatabaseName("studytest")
			.withUsername("studytest")
			.withPassword("studytest");

	@BeforeAll
	static void setUp() {
		container.start();
	}

	@AfterAll
	static void clean() {
		container.stop();
	}
}
```
- 컨테이너를 선언하는 방법은 두가지가 있는데, 인스턴스 필드로 선언할 경우 매번 테스트마다 컨테이너를 새롭게 띄운다.
- 스태틱 필드로 선언할 경우에는 해당 테스트 클래스 전역에서 사용하게 된다.
- 스태틱 필드로 사용할 경우에는 @BeforeAll 과 @AfterAll 을 사용하여 자원 회수작업을 해주어야 한다.

> 당연한 이야기 겠지만... 로컬에 반드시 Docker 가 실행중이어야 한다.
    
