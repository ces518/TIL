# REST API - Spring - REST DOCS API Index 생성하기
#### Index 생성하기
- API 의 진입점을 통해 리소스를 제공. API의 진입점이 필요하다.
- GET /api 요청시 _links 에 api 리소스에 대한 링크들을 제공한다.

#### IndexController 에 대한 TEST 코드 작성
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureRestDocs
@Import(RestDocsConfiguration.class)
@ActiveProfiles("test")
public class IndexControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void index () throws Exception {
        this.mockMvc.perform(get("/api"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$._links.events").exists())
        ;
    }

}
```

#### IndexController 작성
    - rel: events로 event 목록 api 에 대한 링크르 제공하도록 구현
```java
@RestController
public class IndexController {

    @GetMapping("/api")
    public ResourceSupport index () {
        ResourceSupport index = new ResourceSupport();
        index.add(linkTo(EventController.class).withRel("events"));
        return index;
    }
}
```

#### 테스트 결과
- _links.events 링크가 존재하는것을 확인 가능
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /api
       Parameters = {}
          Headers = []
             Body = null
    Session Attrs = {}

Handler:
             Type = me.june.restapi.index.IndexController
           Method = public org.springframework.hateoas.ResourceSupport me.june.restapi.index.IndexController.index()

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/hal+json;charset=UTF-8"]
     Content type = application/hal+json;charset=UTF-8
             Body = {"_links":{"events":{"href":"http://localhost:8080/api/events"}}}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

#### 에러 발생시 인덱스로 가는 링크 제공
- 보통의 웹사이트 이용중 에러 발생시 인덱스 페이지로 이동하는 링크 정보를 제공한다.
- API 에서도 에러 발생시 인덱스로 이동가능한 링크 정보를 제공 해야한다.
- Errors 객체도 Resource 로 만들어 링크 정보를 추가하는 형태로 제공

#### ErrorResource
- EventResource와 동일하게 Resource<T>를 구현하는 클래스를 생성
- rel: index 로 index 링크 정보를 제공한다.
```java 
public class ErrorResource extends Resource<Errors> {

    public ErrorResource(Errors content, Link... links) {
        super(content, links);
        /* Error Resource 로 변경하여 index에 대한 링크정보를 함께 제공*/
        add(linkTo(methodOn(IndexController.class).index()).withRel("index"));
    }
}
```

#### 이벤트 생성 API 수정
- badRequest 응답시 errors 가 아닌 ErrorResource를 제공한다.
- ErrorResource에는 index에 대한 링크정보가 들어있다.
```java
@PostMapping
public ResponseEntity createEvent (@Valid @RequestBody EventDto eventDto, Errors errors) { // 입력값을 EventDto를 활용하여 받는다.
    eventValidator.validate(eventDto, errors);
    if (errors.hasErrors()) {
        return ResponseEntity.badRequest().body(new ErrorResource(errors));
    }

    Event event = objectMapper.convertValue(eventDto, Event.class);
    // 비즈니스 로직을 수행
    event.update();

    Event savedEvent = eventRepository.save(event);
    // created를 생성할때는 항상 uri를 제공해야한다.
    // org.springframework.hateoas.mvc.ControllerLinkBuilder 를 사용하면 손쉽게 URI를 생성할 수 있음.
    ControllerLinkBuilder linkBuilder = linkTo(EventController.class).slash(savedEvent.getId()); // 새로 생성된 Event의 ID를 기반으로 Location Header로 들어간다.
    URI uri = linkBuilder.toUri();

    /* 링크정보에는 어떤 Method를 사용해야하는지에 대한 정보는 담을 수 없다. */
    EventResource eventResource = new EventResource(savedEvent);
    eventResource.add(linkTo(EventController.class).withRel("query-events"));
    // eventResource.add(linkBuilder.withSelfRel());
    eventResource.add(linkBuilder.withRel("update-event"));
    // profile Link 추가
    eventResource.add(new Link("/docs/index.html#resources-events-create").withRel("profile"));
    return ResponseEntity.created(uri).body(eventResource);
}
```

#### 잘못된 요청시 에러 정보와 함께 인덱스 링크 정보 여부 테스트 
```java
@Test
@TestDescription("이벤트 생성 시작일이 종료일을 넘을경우 테스트")
public void 이벤트생성_시작일이_종료일을_넘을경우_BAD_REQUEST () throws Exception {
    // 입력값을 아무것도 보내지않을 경우 테스트
    EventDto eventDto = EventDto.builder()
            .name("Spring")
            .description("REST API Study")
            .beginEnrollmentDateTime(LocalDateTime.of(2019, 10 , 5, 11, 23))
            .closeEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
            .beginEventDateTime(LocalDateTime.of(2019, 10, 15, 14, 21))
            .endEventDateTime(LocalDateTime.of(2019, 8, 16, 14, 21))
            .basePrice(100)
            .maxPrice(200)
            .limitOfEnrollment(100)
            .location("대전 둔산동 스타벅스")
            .build();

    this.mockMvc.perform(post("/api/events")
            .contentType(MediaType.APPLICATION_JSON_UTF8)
            .accept(MediaTypes.HAL_JSON_UTF8)
            .content(objectMapper.writeValueAsString(eventDto)))
            .andDo(print())
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$[0].objectName").exists())
            .andExpect(jsonPath("$[0].field").exists())
            .andExpect(jsonPath("$[0].defaultMessage").exists())
            .andExpect(jsonPath("$[0].code").exists())
            .andExpect(jsonPath("$[0].rejectedValue").exists())
            .andExpect(jsonPath("$._links.index").exists())
    ;
}
```

#### 테스트 결과
- 왜 테스트가 실패할까 ?
- json 응답에 Unwrap 되지 않았다.
- ResourceSupport 를 구현하는 하위 클래스중 Resource<T> 를 사용하면 <T> 에 해당하는 Resource가 content 필드로 제공된다.
- getContent() method 에는 @JsonUnwrapped 애노테이션이 붙어있을텐데 왜 Unwrap되지 않았을까 ?
```java
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/hal+json;charset=UTF-8"]
             Body = {"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-10-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-10-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100}
    Session Attrs = {}

Handler:
             Type = me.june.restapi.events.EventController
           Method = public org.springframework.http.ResponseEntity me.june.restapi.events.EventController.createEvent(me.june.restapi.events.EventDto,org.springframework.validation.Errors)

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 400
    Error message = null
          Headers = [Content-Type:"application/hal+json;charset=UTF-8"]
     Content type = application/hal+json;charset=UTF-8
             Body = {"content":[{"field":"endEventDateTime","objectName":"eventDto","code":"wrongValue","defaultMessage":"EndEventDateTime is must be more than begin","rejectedValue":"2019-08-16T14:21"}],"_links":{"index":{"href":"http://localhost:8080/api"}}}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []



java.lang.AssertionError: No value at JSON path "$[0].objectName"
```

#### JSON Array 는 Unwarp 되지 않는다.
- @JsonUnwrapped 애노테이션 상단의 주석을 보면 JSON Array는 Unwarp 할수 없다.
```java
 * Annotation can only be added to properties, and not classes, as it is contextual.
 *<p>
 * Also note that annotation only applies if
 *<ul>
 * <li>Value is serialized as JSON Object (can not unwrap JSON arrays using this
 *   mechanism)
 *   </li>
 * <li>Serialization is done using <code>BeanSerializer</code>, not a custom serializer
 *   </li>
 * <li>No type information is added; if type information needs to be added, structure can
 *   not be altered regardless of inclusion strategy; so annotation is basically ignored.
 *   </li>
 * </ul>
 */
@Target({ElementType.ANNOTATION_TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotation
public @interface JsonUnwrapped
```
