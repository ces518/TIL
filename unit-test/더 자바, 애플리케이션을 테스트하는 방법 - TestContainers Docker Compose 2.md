# 더 자바, 애플리케이션을 테스트하는 방법

## TestContainers 도커 Compose 사용 - 2
- docker compose 서비스 정보 참조

```java
@Container
static DockerComposeContainer composeContainer =
        new DockerComposeContainer(new File("src/test/resources/docker-compose.yml"))
        .withExposedService("study-db", 5432);

static class ContainerPropertyInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {

	@Override
	public void initialize(ConfigurableApplicationContext context) {
		TestPropertyValues.of("container.port=" + composeContainer.getServicePort("study-db", 5432))
				.applyTo(context.getEnvironment());
	}
}
```
