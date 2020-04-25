
#### Spring Boot 테스트

##### 의존성 추가
- 스프링 부트 애플리케이션을 테스트 하기 위해서는 spring-boot-starter-test 의존성을 아래와 같이 추가해주어야 한다.

`Maven`
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

`기본적인 테스트 클래스의 형태`
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
public class HelloControllerTest {

}
```

> Junit5 의 경우에는 @RunWith 애노테이션이 생략된다.

`webEnvironment 속성`
- MOCK: 테스트 환경에서 내장 콤켓을 구동하지 않는다 (Mock up 된 환경에서 테스트)
- RANDOM_PORT_DEFINED_PORT: 내장 톰캣을 사용한다
- NONE: 서블릿 환경을 제공하지 않는다.

> 기본값은 MOCK 으로 되어있어, 서브릿 컨테이너가 아닌 Mock up 된 서블릿을 실행한다. 디스페처 서블릿에 요청을 보내는것과 유사하게 테스트를 할 수 있다.
> mcok-up된 서블릿에 요청을 보내기 위해선 MockMVc 객체를 활용해야 한다.

#### MockMvc
- MockMvc 를 사용하는 방법은 여러가지가 있는데 SpringBoot 환경에서는 @AutoConfigureMockMvc 애노테이션을 사용하면 손쉽게 설정이 완료된다.
- 또한 @Autowired 애노테이션을 통해 MockMVc 객체를 주입받을 수 있다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class HelloControllerTest{

    @Autowired
    MockMvc mockMvc;

}
```

#### RestTemplate
- RANDON_PORT나 DEFINED_PORT 옵션을 사용할경우 서블릿 컨테이너인 내장 톰캣으 구동된다.
- 이럴 경우 RestTemplate 혹은 WebClient 를 사용해야 한다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class HelloControllerTest{

    @Autowired
    TestRestTemplate testRestTemplate;

}
```

> TestRestTemplate 을 사용할 경우 문제는 테스트 환경이 너무 커진다는 것이다.
> 슬라이싱 테스트만 작성하고 싶은데, 서비스가 관여하게 된다. 서비스의 완전한 구현체가 반드시 존재해야 한다.
> 서비스 객체를 Mocking 하여 슬라이싱 테스트를 진행하고 싶을때에는 @MockBean 애노테이션을 사용하여 Mocking 할 수 있다.

```java
@MockBean
HelloService helloService;

@Test
public void hello() {
    when(helloService.hello()).thenReturn("hello");

    // test code ...
}
```

##### WebClient
- SpringMvc WebFlux 에 추가된 rest 클라이언트중 하나이다.
- 비동기 방식으로 동작한다.

`의존성 추가`
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class HelloControllerTest{

    @Autowired
    WebTestClient webTestClient;
}
```

##### @SpringBootTest
`@SpringBootTest 간단한 동작 방식`
- SpringBootApplication 을 찾아, 테스트용 ApplicationContext를 만들며 빈을 자동등록 한다.
- 이후 MockBean을 찾아 교체한다.

> MockBean 은 테스트마다 리셋되므로 관리할 필요가 없다.

##### 슬라이싱 테스트
- 레이어별로 슬라이싱 테스트 기능을 제공한다.

`@JsonTest`
- json 응답만 테스트

`@WebMvcTest`
- 컨트롤러 슬라이싱 테스트시 주로 사용된다.
- 웹과 관련된 빈 (컨트롤러, 필터 등) 만 등록된다.

`@WebFluxTest`
- SpringMvc WebFlux 관련 테스트

`@DataJpaTest`
- Spring data JPA 사용시 Repository 계층만 테스트할 때 사용된다.

##### 테스트 유틸
- OutputCapture
- TestPropertyValues
- TestRestTemplate
- ConfigFileApplicationContextInitializer

> OutputCapture은 JUnit의 Rule를 확장해서 만들어졌다. OutputCapture는 @Rule의 제약사항때문에 public 접근제어자를 지정해야한다. OutputCapture를 이용하면 로그 메시지를 이용한 테스트를 작성하기가 쉬워진다. 로그 메시지를 중간 중간 중요한 부분에 출력하는 코드를 작성하고, 해당 로그가 출력되었는지 테스트할 수 있다.
