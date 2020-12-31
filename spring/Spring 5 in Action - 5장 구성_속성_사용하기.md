# Spring 5 in Action

## 5장 구성 속성 사용하기

### 자동 구성 세부 조정하기
- 스프링에는 두 가지 형태의 서로 다르면서도 관련이 있는 구성이 있다.
1. **빈 연결 (Bean wiring)** 
  - 스프링 애플리케이션 컨텍스트에서 빈으로 생성되는 애플리케이션 컴포넌트 및 상호 간에 주입되는 방법을 선언하는 구성
2. **속성 주입 (Property wiring)**
  - 스프링 애플리케이션 컨텍스트에서 빈의 속성 값을 설정하는 구성
    
- 위 두가지 구성은 스프링의 XML 구성과 자바 기반 구성 모두에서 선언된다.

#### 스프링 환경 추상화
- **스프링 환경 추상화 (environment abstraction)** 는 구성가능한 모든 속성을 **한곳** 에서 관리하는 개념이다.
- 속성의 근원을 추상화 하여 필요로 하는 빈이 스프링 자체에서 해당 속성을 사용할 수 있게 해준다.
- 스프링 환경에서는 아래 4가지를 지원한다.
1. JVM 시스템 속성
2. 운영체제 환경 변수
3. 명령행 인자 
4. 애플리케이션 속성 구성 파일

`프로퍼티 우선순위`
1. 홈 디렉토리에 존재하는 spring-boot-dev-tools.properties
2. 테스트에 존재하는 @TestPropertySource
3. @SpringBootTest 애노테이션의 properties 애트리뷰트
4. 커맨트라인 아규먼트
5. SPRING_APPLICATION_JSON (환경변수 혹은 시스템 프로퍼티) 에 존재하는 프로퍼티
6. ServletConfig 파라메터
7. ServletContext 파라메터
8. java:comp/env JNDI 애트리뷰트
9. System.getProperty() 자바 시스템 프로퍼티
10. OS 환경변수
11. RandomValuePropertySource
12. JAR 밖에 존재하는 특정 프로파일의 application.properties
13. JAR 안에 존재하는 특정 프로파일의 application.properties
14. JAR 밖에 존재하는 application.properties
15. JAR 안에 존재하는 application.properties
16. @PropertySource
17. 기본 프로퍼티

- 예를 들어 애플리케이션을 실행 시켜주는 서블릿 컨테이너의 8080 기본 포트를 변경하고 싶다면 다음과 같이 할 수 있다.

`properties`
```properties
server.port=9090
```

`yaml`
```yaml
server:
  port: 9090
```

`JVM Options`
```shell
java -jar tacocloud-0.0.5-SNAPSHOT.jar --server.port=9090
```

`OS Enviroment`
```shell
export SERVER_PORT=9090
```

> 위 4가지 모두 동작하며, OS 환경 변수로 속성을 지정할 때는 운영체제의 환경 변수 네이밍 규칙이 있기 때문에 조금 다르지만
> 스프링에서 SERVER_PORT 를 server.port 로 인식하기 때문에 문제가 되지 않는다.

#### 데이터 소스 구성
- 현재 H2 데이터베이스를 사용하고 있는데, 지금은 우리의 요구사항을 충족시켜 준다.
- 하지만 프로덕션 환경에서 사용하게 된다면 더 확실한 데이터베이스 솔루션을 원하게 될것이다.
- 그런 경우 우리 나름의 DataSource 빈을 명시적을 구성할수도 있지만, 스프링 부트는 구성 속성을 통해 설정하는 것이 더 간단하다.

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost/tacocloud
    username: tacouser
    password: tacopassword
    driver-class-name: com.mysql.jdbc.Driver
```
- 스프링 부트는 DataSource 를 자동구성할 때 JDBC 커넥션풀도 함께 설정하는데, 만약 classpath 에 톰캣 JDBC 커넥션풀이 존재한다면 그것을 사용한다.
- 만약 존재하지 않을경우 아래의 커넥션풀을 찾아 사용하게 된다.
  - HikariCP
  - Commons DBCP2

> Commons DBCP 와 Tomcat DBCP 는 동작 방식이 유사하다. (내부적으로 락을 잡거나, Concurrent 패키지 라이브러리를 활용하는 등의 차이)
> 최근에는 **HikariCP** 를 가장 많이 사용하며, 성능이 매우 매우 좋다.

#### 내장 서버 구성
- server.port 속성의 값을 0으로 지정하면 서버는 0번 포트로 시작되는 것이 아니다.
- 사용가능한 포트를 무작위로 선택해서 시작하게 된다.
- 이는 자동화된 통합테스트 혹은 마이크로서비스와 같이 애플리케이션 시작 포트가 중요하지 않을 때 유용하다.

#### 로깅 구성
- 스프링 부트는 Commons Logging 을 사용한다.
  - 스프링 코어에서 Commons Logging 을 사용하기 때문
- SLF4j 를 사용하면 의존성 설정을 잘 해주어야 한다..
- 기본적으로 스프링 부트는 INFO LEVEL 로 로깅을 한다.
- 보통 logback.xml 을 통해 구성을 제엉하지만, 스프링 부트의 구성속성을 사용하면 해당 파일 없이도 로깅 수준을 변경할 수 있다.

```yaml
logging:
  path: /var/logs/
  file: TacoCloud.log
  level:
    root: warn
  
```

`로깅 퍼사드와 로거`
- Commons Logging, SLF4j
  - 실제 로깅을 하는 구현체가 아닌 추상화 해놓은 API 이다.
  - 프레임워크 들은 로깅 퍼사드를 이용하여 개발을 진행한다.
  - 실제 구현체들을 교체할 수 있도록 함이 그 이유이다.
  - JUL, Log4j2, Logback 등이 실제 구현체
- Spring-JCL
  - Commons-Logging -> SLF4j 로 변경할 수 있도록 제공 한다.

> Commons Logging -> SLF4j -> Logback 
> 스프링 부트는 logback 을 실제 구현체로 사용하게 된다.

### 커스텀 구성속성 생성
- 구성 속성은 빈의 속성일 뿐이며, 스프링 환경 추상화로부터 여러가지 구성을 받기위해 설계 되었다.
- 구성 속성의 올바른 주입을 지원하기 위해 스프링 부트는 @ConfigurationProperties 애노테이션을 제공한다.
- 이는 어떤 스프링빈이던, 해당 빈의 속성들이 스프링 환경의 속성으로 부터 주입될 수 있다.

`application.yml`
```yaml
taco:
  orders:
    pageSize: 10
```

`OrderProps`
```java
@Component
@ConfigurationProperties(prefix = "taco.orders")
@Data
public class OrderProps {
    private int pageSize = 20;
}
```

### 프로파일을 활용한 구성
- 스프링에서는 스프링 프로파일 기능을 제공한다.
- 런타임 시에 활성화 되는 프로파일에 따라 서로 다른빈, 구성 클래스, 구성 속성들이 적용 혹은 무시되도록 하는 기능이다.

#### 프로파일 특정 속성 정의
- 특정 속성을 정의하는 방법중 한가지는 특정 환경의 속성들만 포함하는 yml 혹은 properties 파일을 생성하는 것이다.
- 이는 네이밍 규칙이 있으며 규칙은 다음과 같다.
- application-{프로파일명}.yml | .properties
- 다음은 prod 프로파일시 사용되는 설정파일의 예시이다.

`application-prod.yaml`
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost/tacocloud
    username: tacouser
    password: tacopassword
logging:
  level:
    tacos: warn

```

- 추가적으로 YAML 구성에서만 가능한 방법도 존재하는데 하나의 yml 파일에 여러 프로파일을 정의할 수 있다.
`application.yml`
```yaml
logging:
  level:
    tacos: debug

---
spring:
  profiles: prod
```

- 프로파일 활성화 시에는 profiles.active 속성에 명시해 주면 다음과 같이 프로파일 활성화가 적용 된다.
```yaml
spring:
  profiles:
    active:
    -prod
```

> 이는 환경 변수, JVM Options 로 지정할 수도 있다.

#### 프로파일을 사용한 조건별 빈 생성
- 특정 프로파일에는 특정 빈을 등록한다거나, 하는 경우가 필요할 수가 있다.
- 이때 @Profile 애노테이션을 지정하면, 특졍 프로파일 환경에서는 해당 빈이 등록되거나, 혹은 등록에서 제외되거나 하는등 설정이 가능해진다.
- 아래 예제는 dev, qa 프로파일에만 해당 빈이 등록되도록 하는 설정이다.

```java
@Bean
@Profile({"dev", "qa"})
public ApplicationRunner dataLoader(IngredientJpaRepository repository) {
  return args -> {
  repository.save(new Ingredient("FLTO", "Flour Tortilla", Ingredient.Type.WRAP));
  repository.save(new Ingredient("COTO", "Corn Tortilla", Ingredient.Type.WRAP));
  repository.save(new Ingredient("GRBF", "Ground Beef", Ingredient.Type.PROTEIN));
  repository.save(new Ingredient("CARN", "Carnitas", Ingredient.Type.PROTEIN));
  repository.save(new Ingredient("TMTO", "Diced Tomatoes", Ingredient.Type.VEGGIES));
  repository.save(new Ingredient("LETC", "Lettuce", Ingredient.Type.VEGGIES));
  repository.save(new Ingredient("CHED", "Cheddar", Ingredient.Type.CHEESE));
  repository.save(new Ingredient("JACK", "Monterrey Jack", Ingredient.Type.CHEESE));
  repository.save(new Ingredient("SLSA", "Salsa", Ingredient.Type.SAUCE));
  repository.save(new Ingredient("SRCR", "Sour Cream", Ingredient.Type.SAUCE));
  };
}
```

## 정리
- 스프링 빈에 @ConfigurationProperties 를 사용하면 구성 속성의 값을 주입할 수 있다.
- 구성 속성은 커맨드라인 아규먼트, 환경 변수, JVM 옵션, 속성 파일 등 에서 설정할 수 있다.
- 스프리링 프로파일은 활성화된 프로파일을 기반으로 구성 속성을 설정하기 위해 사용할 수 있다.