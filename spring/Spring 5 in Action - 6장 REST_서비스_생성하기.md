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
  - http://homoefficio.github.io/2019/10/03/Java-Optional-바르게-쓰기/

- **ResponseEntity** 는 응답 헤더의 상태코드와 본문을 직접 다루고 싶은경우 사용한다.
  - 자주 사용하는 응답의 경우 static factory method 를 제공한다.
  - 이 자체가 응답본문이기 때문에 @ResponseBody 를 한것과 같다. 
  - REST API 라면, 이는 선택이 아닌 필수라고 생각한다. -> 명확한 HTTP 응답 코드가 필요하다.
  - HTTP 응답 코드에 대한 고찰.. 만약 권한이 없는 사용자가 특정 리소스에 대해 접근을 시도했을때 401 ? 404 ? 이런 고민이 필요하다.

#### PUT 과 PATCH

| Resource | GET | PUT | POST | DELETE |
|---|---|---|---|---|
| http://example.com/resources/ | 컬렉션에 속한 자원들의 URI 혹은 상세 목록을 보여준다. | 전체 컬렉션을 다른 컬렉션으로 교체한다. | 해당 컬렉션에 속한느 새롱누 자원을 생성한다. | 전체 컬렉션을 삭제한다.
| http://example.com/resources/1 | 요청한 컬렉션 내의 자원을 반환한다. | 해당 자원을 수정한다. | 새로운 자원을 생성한다. | 해당 컬렉션 내 자원을 삭제한다. |

- **PATCH** 라는 메소드에 유의 해야 한다.
- PUT 은 해당 자원의 전체를 교체한다는 의미이고, PATCH 는 일부를 변경한다는 의미를 지닌다.
- 최근에는 UPDATE 이벤트시 PUT 보다는 PATCH 가 더 의미로 적합하다고 평가되고 있따.
- Rails 4.0 부터는 PATCH 가 UPDATE 이벤트의 기본 Method 로 사용될것이다.

- https://spoqa.github.io/2012/02/27/rest-introduction.html

### HATEOAS
- **HATEOAS (Hypermedia As The Engine Of Application State)** 는 REST API 를 구현하는 방법중 하나
- API 로 부터 반환되는 리소스에 관련된 하이퍼 링크들을 포함하는 방법
- REST 아키텍쳐의 조건중 **Uniform Interface** 조건..
- HATEOAS 를 구현하는 방법 ? -> **HAL**
- **HAL (Hypertext Application Language)** 은 JSON 응답에 하이퍼링크를 포함시킬때 주로 사용하는 형식
    - 중요한것은 -> Resource 와 Link
- https://velog.io/@pop8682/번역-HAL-Hypertext-Application-Language

`하이퍼 링크를 포함한 응답`
```json
{
  "_embedded": {
    "eventList": [
      {
        "id": 27,
        "name": "event26",
        "description": "test26",
        "beginEnrollmentDateTime": null,
        "closeEnrollmentDateTime": null,
        "beginEventDateTime": null,
        "endEventDateTime": null,
        "location": null,
        "basePrice": 0,
        "maxPrice": 0,
        "limitOfEnrollment": 0,
        "offline": false,
        "free": false,
        "eventStatus": null,
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/events/27"
          }
        }
      },
      {
        "id": 26,
        "name": "event25",
        "description": "test25",
        "beginEnrollmentDateTime": null,
        "closeEnrollmentDateTime": null,
        "beginEventDateTime": null,
        "endEventDateTime": null,
        "location": null,
        "basePrice": 0,
        "maxPrice": 0,
        "limitOfEnrollment": 0,
        "offline": false,
        "free": false,
        "eventStatus": null,
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/events/26"
          }
        }
      }
    ]
  },
  "_links": {
    "first": {
      "href": "http://localhost:8080/api/events?page=0&size=10&sort=name,desc"
    },
    "prev": {
      "href": "http://localhost:8080/api/events?page=0&size=10&sort=name,desc"
    },
    "self": {
      "href": "http://localhost:8080/api/events?page=1&size=10&sort=name,desc"
    },
    "next": {
      "href": "http://localhost:8080/api/events?page=2&size=10&sort=name,desc"
    },
    "last": {
      "href": "http://localhost:8080/api/events?page=2&size=10&sort=name,desc"
    },
    "profile": {
      "href": "/docs/index.html#resources-events-list"
    }
  },
  "page": {
    "size": 10,
    "totalElements": 30,
    "totalPages": 3,
    "number": 1
  }
}
```

### Spring HATEOAS
- HATEOAS 를 매번 직접 구현하는 것은 매우 번거롭고 힘든일이다.
- 이를 위해 Spring HATEOAS 프로젝트가 존재한다. Spring HATEOAS 프로젝트는 하이퍼링크를 스프링에서 제공한다.
- Spring HATEOAS 를 사용하기 위해 아래 의존성을 추가하자.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

- 책에서 나오는 예제들은 모두 HATEOAS 0.25.x 버전 기준이다... 해당 클래스들은 1.x 버전으로 올라오면서 모두 변경되었으니 주의

```java
@RestController
@RequestMapping(path = "/api/design", produces = { MediaType.APPLICATION_JSON_VALUE, MediaType.TEXT_XML_VALUE })
@CrossOrigin(origins = "*") // CrossOrigin 은 무엇인가 ?
@RequiredArgsConstructor
public class DesignTacoApiController {
    private final TacoJpaRepository tacoRepository;

    @GetMapping("/recent")
    public CollectionModel<Taco> recentTacos() {
        PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
        // Spring-HATEOAS 0.25.x 기준 Resource -> 1.x 로 올라오면서 EntityModel, CollectionModel 로 변경됨
        List<Taco> tacos = tacoRepository.findAll(page).getContent();
        return CollectionModel.of(tacos, linkTo(methodOn(DesignTacoApiController.class).recentTacos()).withRel("recents"));
    }
}
```
- 이전 구현과 달라진 점이 있다면, Taco 타입의 CollectionModel 을 반환한다는 것이다.
  - 1.x 로 버전업되면서 Resource 관련 클래스들은 EntityModel, CollectionModel 로 변경되었다.
- Link 정보를 제공하기 위해 linkTo, methodOn 메소드를 사용하였다.
- 이 둘을 조합해서 사용하면 "/api/design/recent" 와 같이 하드코딩 하지않고 타입세이프한 코드가 가능해진다.
- 하지만 매번 이런 작업을 하는것은 번거로운 일이다.

#### RepresentationModel 과 RepresentationModelAssembler
`TacoModel`
```java
// HATEOAS 0.25.x ResourceSupport -> HATEOAS 1.x RepresentationModel 로 변경
@Getter
public class TacoModel extends RepresentationModel<TacoModel> {
    private final String name;
    private final Date createdAt;
    private final CollectionModel<IngredientModel> ingredients;

    public TacoModel(String name, Date createdAt, CollectionModel<IngredientModel> ingredients) {
        this.name = name;
        this.createdAt = createdAt;
        this.ingredients = ingredients;
    }
}
```

`TacoModelAssembler`
```java
@Component
public class TacoModelAssembler extends RepresentationModelAssemblerSupport<Taco, TacoModel> {
    private final IngredientModelAssembler ingredientModelAssembler;

    public TacoModelAssembler(IngredientModelAssembler ingredientModelAssembler) {
        super(DesignTacoApiController.class, TacoModel.class);
        this.ingredientModelAssembler = ingredientModelAssembler;
    }

    @Override
    protected TacoModel instantiateModel(Taco entity) {
        return new TacoModel(entity.getName(), entity.getCreatedAt(), ingredientModelAssembler.toCollectionModel(entity.getIngredients()));
    }

    @Override
    public TacoModel toModel(Taco entity) {
        return createModelWithId(entity.getId(), entity);
    }
}
```

- TacoModel 을 사용하는 이유? -> API 에서 CollectionModel 요소를 반환할때 컬렉션의 요소마다 Link 를 추가해야한다. 이는 번거로운 일이다.
- RepresentationModel 과 RepresentationModelAssembler 는 이런 반복적이고 번거로운 작업을 줄여주는 역할을 한다.
  - HATEOAS 0.25.x ResourceSupport -> HATEOAS 1.x RepresentationModel 로 변경되었다.
- RepresentationModelAssembler 는 RepresentationModel 를 기반으로 링크정보를 생성하는 빌더 역할을 수행한다.
  - 추가적으로 **SimpleRepresentationModelAssembler** 인터페이스도 제공한다.
- 위 구현체들을 적용한 최종 코드는 다음과 같다.

```java
@RestController
@RequestMapping(path = "/api/design", produces = { MediaType.APPLICATION_JSON_VALUE, MediaType.TEXT_XML_VALUE })
@CrossOrigin(origins = "*") // CrossOrigin 은 무엇인가 ?
@RequiredArgsConstructor
public class DesignTacoApiController {
    private final TacoJpaRepository tacoRepository;
    private final TacoModelAssembler tacoModelAssembler;

    @GetMapping("/recent")
    public CollectionModel<TacoModel> recentTacos() {
        PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
        List<Taco> tacos = tacoRepository.findAll(page).getContent();
        return tacoModelAssembler.toCollectionModel(tacos);
    }
}
```

### Spring data REST
- 스프링 데이터 는 인터페이스를 기반으로 리포지토리 구현체를 자동으로 생성해준다.
- 이뿐만이 아닌 애플리케이션의 API 를 정의하는데 도움을 주는 기능도 제공한다.
- Spring data REST 는 스프링 데이터가 생성하는 리포지토리의 REST API 를 자동으로 생성한다.
- Spring data REST 를 사용하기 위해 아래 의존성을 추가해주자.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```

- 의존성만 추가해 주면 REST API 구현이 끝난다.
- Spring data REST 가 생성해주는 REST 엔드 포인트는 우리가 직접 구현한것 만큼 좋다.
- 이를 사용하려면 @RestController 를 통해 직접 구현해주었던 엔드포인트를 제거해 주어야한다.
  - Spring data REST 가 구현해주는 엔드포인트가 우선순위가 더 높다.
- 자동으로 구현된 식자재 api 를 조회해보면 다음과 같이 우리가 구현한 방식과 유사하게 응답을 받을 수 있다.  

```json
{
  "_embedded": {
    "ingredients": [
      {
        "name": "Flour Tortilla",
        "type": "WRAP",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/FLTO"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/FLTO"
          }
        }
      },
      {
        "name": "Corn Tortilla",
        "type": "WRAP",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/COTO"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/COTO"
          }
        }
      },
      {
        "name": "Ground Beef",
        "type": "PROTEIN",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/GRBF"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/GRBF"
          }
        }
      },
      {
        "name": "Carnitas",
        "type": "PROTEIN",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/CARN"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/CARN"
          }
        }
      },
      {
        "name": "Diced Tomatoes",
        "type": "VEGGIES",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/TMTO"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/TMTO"
          }
        }
      },
      {
        "name": "Lettuce",
        "type": "VEGGIES",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/LETC"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/LETC"
          }
        }
      },
      {
        "name": "Cheddar",
        "type": "CHEESE",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/CHED"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/CHED"
          }
        }
      },
      {
        "name": "Monterrey Jack",
        "type": "CHEESE",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/JACK"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/JACK"
          }
        }
      },
      {
        "name": "Salsa",
        "type": "SAUCE",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/SLSA"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/SLSA"
          }
        }
      },
      {
        "name": "Sour Cream",
        "type": "SAUCE",
        "_links": {
          "self": {
            "href": "http://localhost:8080/api/ingredients/SRCR"
          },
          "ingredient": {
            "href": "http://localhost:8080/api/ingredients/SRCR"
          }
        }
      }
    ]
  },
  "_links": {
    "self": {
      "href": "http://localhost:8080/api/ingredients"
    },
    "profile": {
      "href": "http://localhost:8080/api/profile/ingredients"
    }
  }
}
```
> spring data rest 가 생성한 엔드포인트의 base-path 를 지정하려면 spring.data.rest.base-path 속성을 지정해 주면 된다.
> 그렇다면 모든 경로에 base-path 가 적용된 엔드포인트가 생성된다.

#### 주의점
- 자동 생성된 API 를 테스트하던 도중 /api/tacos 라고 요청을 하면 404 응답을 받는다.
- 이유 ? -> Spring data REST 는 tacos 라는 엔드포인트도 제공하는데, 이를 노출하는 방법이 문제가 된다.
  - Ingredients : /ingredients
  - Order : /orders
  - User : /users
  - Taco : ???
- Taco 의 엔드포인트는 tacos 가 아닌 tacoes 와 같이 복수형으로 지정을 하기 때문에 404 응답을 받은것.. /api/tacoes 로 요청하면 정상적인 응답을 받을 수 있다.
- 하지만 Taco 만 tacos 가 아닌 tacoes 가 된다면 문제가 될 수도 있다.
- 이런 복수형 관련 문제를 해결하기 위해 Spring data REST 에서는 **@RestResource** 라는 애노테이션을 제공한다.
- 아래와 같이 RestResource 를 적용하면 우리가 원하던 대로 /api/tacos 를 엔드포인트로 사용할 수 있다.

```java
@Data
@Entity
@RestResource(rel = "tacos", path = "tacos") // spring-data-rest 의 relation 네임을 설정함.. tacoes -> tacos 로 변경
public class Taco {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private Date createdAt;

    @NotNull
    @Size(min = 5, message = "Name must be at least 5 characters long")
    private String name;

    @ManyToMany
    @Size(min = 1, message = "You must choose at least 1 ingredient")
    private List<Ingredient> ingredients;

    @PrePersist
    void createdAt() {
        this.createdAt = new Date();
    }
}
```
  
#### 커스텀 엔드포인트 추가하기
- Spring data REST 가 제공하는 엔드포인트 외에도 우리가 직접 작업한 엔드포인트를 추가하고 싶은 경우가 있다.
- @RestController 를 사용한 클래스를 구현하여 API 엔드 포인트에 추가할 수 있지만, 두가지 문제가 있다.

1. @RestController 를 사용하여 구현할 수는 있지만, Spring data REST 의 base-path 는 적용되지 않는다.
2. Spring data REST 의 응답에서 제공하는 하이퍼링스에 자동으로 포함되지 않는다.

- 위 두가지 문제를 해결하기 위해 스프링 데이터 REST 는 **@RepositoryRestController** 애노테이션을 제공한다.
- @RepositoryRestController 를 사용한 구현체는 첫번재 문제를 해결해 준다.
- 두번째 문제를 해결하기 위해서는 또 다른 방법이 필요하다.

#### 커스텀 하이퍼링크를 엔드포인트에 추가하기
- 두번째 문제를 해결하기 위해서는 **RepresentationModelProcessor** 를 구현해야 한다.
- 이는 API 를 통해 리소스가 반환되기 전 리소스를 조작하는 인터페이스 이다.
- 주의할 점은 우리가 이전에 구현했던 **TacoModel** 클래스 타입을 반환하면 동작하지 않는다. 
  - 이유 ? -> TacoModel 은 우리 직접구현할때만 동작을 한다. Spring-data-REST 는 **Spring-data** 라는 점에 유의 해야한다. Entity 기반으로 동작함!!

```java
@Component
public class TacoModelProcessor implements RepresentationModelProcessor<CollectionModel<Taco>> {

    @Override
    public CollectionModel<Taco> process(CollectionModel<Taco> model) {
        return model.add(
                linkTo(methodOn(RecentTacosController.class).recentTacos()).withRel("recent").expand(model)
        );
    }
}
```

## 정리
- REST 엔드 포인트는 Spring MVC 와 동일한 모델을 따르는 컨트롤러로 생성할 수 있다.
- @ResponseBody 혹은 ResponseEntity 를 사용하면 메소드의 반환값을 응답 본문으로 사용한다.
- @RestController 애노테이션을 사용하면 모든 메소드에 @ResponseBody 를 사용한 것과 동일하다.
- Spring HATEOAS 는 리소스의 하이퍼링크를 쉽게 추가할수 있게 해준다.
- Spring data REST 는 Spring data Repository 를 기반으로 REST API 를 자동 생성 해준다.