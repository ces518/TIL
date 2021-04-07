# 더 자바, 애플리케이션을 테스트하는 방법

## TestContainers 도커 Compose 사용 - 1
- 테스트에서 여러 컨테이너를 사용해야 할때 docker-compose 와 TestContainers 를 연동할 수 있다.

- Docker Compose
  - https://docs.docker.com/compose/
    - 여러 컨테이너를 한번에 띄우고 서로 간의 의존성 및 네트워크 등을 설정할 수 있는 방법
    - docker-compose up / down

- Testcontainser의 docker compose 모듈을 사용할 수 있다.
    - https://www.testcontainers.org/modules/docker_compose/

- 대체제 (1.12.3 버전 기준으로 제대로 실행이 되질 않아 사용한 대체제)
    - https://github.com/palantir/docker-compose-rule
    - 2019 가을 KSUG 발표 자료 참고
    - https://bit.ly/2q8S3Qo


`docker-compose.yml`

```yaml
version: "3"

services:
  study-db:
    image: postgres
    ports:
      - 5432
    environment:
      POSTGRES_PASSWORD: study
      POSTGRES_USER: study
      POSTGRES_DB: study
```
    
```java

@SpringBootTest
@ExtendWith(MockitoExtension.class)
@ActiveProfiles("test")
@Testcontainers
@Slf4j
class StudyServiceTest {

	@Container
	static DockerComposeContainer container =
			new DockerComposeContainer(new File("src/test/resources/docker-compose.yml"));
	// docker-compose 에 서비스가 많다면, 컨테이너 초기화가 다 이뤄지지 않았음애도 테스트가 실행되는 문제가 있음
	// Wait 기능을 사용해서 어느정도 대기를 해주어야 한다.
}
```
- 다른 Container 들과 사용 방법이 유사하며 주의할점은 docker-compose 에 정의된 서비스가 많다면
- 컨테이너 초기화가 다 이루어지지 않았음에도 테스트가 실행되는 문제가 발생할 수 있다.
- 이때 Wait 기능을 사용해서 컨테이너 초기화를 대기했다가 실행하는 조치가 필요하다.