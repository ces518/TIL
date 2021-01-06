# Spring 5 in Action

## 6장 REST 서비스 생성하기

### REST 란 무엇인가 ?
`위키 백과`
> **REST(Representational State Transfer)** 는 월드 와이드 웹과 같은 분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식이다. 
> 이 용어는 **로이 필딩(Roy Fielding)** 의 2000년 박사학위 논문에서 소개되었다.
>
> REST 는 **네트워크 아키텍쳐 원리**의 모음이다.
> - 자원을 정의하고, 자원에 대한 주소를 지정하는 방법 전반을 일컫는다.

**RE** presentational **S** tate **Transfer** 의 약자이며 인터넷 상 시스템간의 상호 운용성을 제공하는 방법중 하나이다.

웹을 깨트리지 않으면서 HTTP 를 진화시키는 방법

시스템 제 각각의 **독립적인 진화를 보장** 하기 위한 방법

- https://ko.wikipedia.org/wiki/REST
- https://github.com/ces518/TIL/blob/master/restapi/RESTfulAPI.md

### REST 컨트롤러 작성하기
- 요즘에는 REST API 와 **SPA (Single-Page Application)** 의 조합으로 구성을 많이 하게된다.
- 2장 에서는 Spring MVC 를 사용한 전통적인 **MPA (Multi-Page Application)** 을 구성했다.
- SPA 클라이언트 코드는 HTTP 를 기반으로 REST API 로 통신한다.
- 스프링은 REST 컨트롤러를 위한 다양한 애노테이션들을 제공한다.

`타코 디자인 API 를 처리하는 REST 컨트롤러`
```java
@RestController
@RequestMapping(path = "/api/design", produces = { MediaType.APPLICATION_JSON_VALUE, MediaType.TEXT_XML_VALUE })
@CrossOrigin(origins = "*")
@RequiredArgsConstructor
public class DesignTacoApiController {
    private final TacoJpaRepository tacoRepository;
    private final TacoModelAssembler tacoModelAssembler;

    @GetMapping("/recent")
    public List<Taco> recentTacos() {
        PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
        return tacoRepository.findAll(page).getContent();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Taco> tacoById(@PathVariable("id") Long id) {
      Taco taco = tacoRepository.findById(id).orElse(null); // Optional 에 관한 고찰. 코.다.기 내용 참조
      if (taco != null) {
        return ResponseEntity.ok(taco);
      }
      return ResponseEntity.notFound().build();
    }
}
```
- **@RestController** 애노테이션은 해당 클래스가 REST 요청을 처리하는 클래스임을 선언하는 **메타 애노테이션** 이다.
- @RestController 애노테이션의 선언을 살펴보면, @Controller 와 **@ResponseBody** 가 선언되어 있음을 알 수 있다.

`@RestController.java`
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {
    @AliasFor(
        annotation = Controller.class
    )
    String value() default "";
}
```
- **@ResponseBody** 애노테이션은, 해당 요청 처리 메소드에서 반환하는 값을 HTTP 응답 본문으로 사용한다는 선언이다.
  - 해당 Method의 리턴 값을 **HttpMessageConverter** 를 사용하여 **응답본문** 으로 보낸다.
  - accept-header 정보를 참조하여 적절한 MessageConverter를 사용한다.
    - JSON 이라면 Jackson2HttpMessageConverter...
  - @RestController 를 사용할경우, 모든 Method에 @ResponseBody가 있는것과 동일하기 때문에 생략이 가능하다.

- **HttpMessageConverter**
  - HTTP 요청 본문을 객체로 변환하거나, 객체를 HTTP 응답본문으로 변환할 때 사용한다.
  - HttpConverter 의 종료에는 여러가지가 있고, 어떤 요청을 받고, 어떤 응답을 내보낼지에 따라 사용하는 메시지 컨버터가 **달라진다.**

- **Jackson2HttpMessageConverter**
  - JSON 요청 과 응답시 사용하는 MessageConverter
  - Spring boot starter web 을 사용하면 Jackson2ObjectMapper가 classpath 에 존재한다.
    - ObjectMapper가 Bean으로 등록된다.
  - Jackson2HttpMessageConverter가 자동적으로 등록된다.

- 부록 -> **ContentNegotiationViewResolver**

- **@CrossOrigin** 애노테이션은 **CORS (Cross Origin Resource Sharing)** 권한을 부여하도록 선언한다.

`CORS 란 ?`
- 교차 출처 리소스 공유(Cross-Origin Resource Sharing, CORS)는 추가 HTTP 헤더를 사용하여 
- 한 출처에서 실행 중인 웹 애플리케이션이 다른 출처의 선택한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 체제
- 웹 애플리케이션은 리소스가 자신의 출처 (도메인, 프로토콜, 포트) 와 다를 경우 교차 출처 HTTP 요청을 실행한다.
- **보안** 상의 이유로, 브라우저는 스크립트에서 시작한 교차 출처 HTTP 요청을 제한한다.
  - https://developer.mozilla.org/ko/docs/Web/HTTP/CORS 
  - http://www.igloosec.co.kr/BLOG_HTML5%20보안%20취약점?searchItem=&searchWord=&bbsCateId=0&gotoPage=10
  - https://802.11ac.net/2018/12/16/cors/
  
- **@PathVariable** 애노테이션은 요청 URI 패턴의 일부를 핸들러 메서드의 Arguments 로 받아올 수 있다.
    - 타입변환 지원
    - java1.8 부터 Optional 을 지원한다. (required false 와 동일)
    - required 속성을 통해 필수 값 유무를 지정가능 기본은 true
    - @PathVariable 은 어떻게 동작하는가 ? -> PathVariableMethodArgumentResolver

- Optional 에 관한 고찰..

