# REST API - Event 생성 API 구현 - 입력값 제한
#### 입력값 제한
    - ID 혹은 입력받은 데이터로 부터 계산해야하는 값들은 입력을 받지 않아야한다.
    - 애노테이션 을 활용한다.
    - DTO를 생성하여 활용한다.

- DTO를 생성하여 활용하는 방법을 선택
    - 이유 ?
        - 도메인 클래스에 애노테이션이 많아 진다.
        - 클래스가 지저분해지고 뭐가 뭔지 알아보기 힘들어진다.
    - 단점 ?
        - 중복이 발생한다.

- EventDTO
```java
@Getter @Setter
@Builder @AllArgsConstructor @NoArgsConstructor
public class EventDto {
    private String name;

    private String description;

    private LocalDateTime beginEnrollmentDateTime;

    private LocalDateTime closeEnrollmentDateTime;

    private LocalDateTime beginEventDateTime;

    private LocalDateTime endEventDateTime;

    private String location; // optional 없다면 온라인모임

    private int basePrice;

    private int maxPrice;

    private int limitOfEnrollment;
}
```

#### Event 생성 API
    - EventDto를 활용하여 입력값을 제한하여 데이터를 받아온다.
    - ObjectMapper로 EventDto를 EventEntity로 변환
    - 변환된 EventEntity로 save .. 
```java
@RestController
@RequestMapping(value = "/api/events", produces = MediaTypes.HAL_JSON_UTF8_VALUE)
public class EventController {

    private final ObjectMapper objectMapper;
    private final EventRepository eventRepository;

    public EventController(ObjectMapper objectMapper, EventRepository eventRepository) {
        this.objectMapper = objectMapper;
        this.eventRepository = eventRepository;
    }

    @PostMapping
    public ResponseEntity createEvent (@RequestBody EventDto eventDto) { // 입력값을 EventDto를 활용하여 받는다.

        // objectMapper를 활용하여 Event 엔티티를 생성한다.
        Event event = objectMapper.convertValue(eventDto, Event.class);

        Event savedEvent = eventRepository.save(event);
        // created를 생성할때는 항상 uri를 제공해야한다.
        // org.springframework.hateoas.mvc.ControllerLinkBuilder 를 사용하면 손쉽게 URI를 생성할 수 있음.
        ControllerLinkBuilder linkBuilder = linkTo(EventController.class).slash(savedEvent.getId()); // 새로 생성된 Event의 ID를 기반으로 Location Header로 들어간다.
        URI uri = linkBuilder.toUri();
        return ResponseEntity.created(uri).body(savedEvent);
    }
}
```

#### 기존의 테스트코드가 깨진다 ?!
- 테스트코드가 에러가발생
    - NullPointerException 발생
    - ControllerLinkBuilder linkBuilder = linkTo(EventController.class).slash(savedEvent.getId());
- 왜 에러가 났을까 ?
    - Mocking이 적용되지 않는다.
    - 파라메터로 Event객체를 그대로 사용하지 않고, HandlerMethod 내부에서 새롭게 생성한 객체이기 때문에 Mocking이 제대로 되지 않는다.
- 테스트코드가 에러가발생
    - Mocking이 적용되지않음
    - event객체를 전달한 객체를 그대로 리턴하지않고 핸들러 메서드에서 새로 생성한 객체이기 때문에 Mocking이 제대로 되지 않음
    - save() 메서드로 전달된 객체는 Handler Method 내부에서 새로 만들어진 객체이다 !!
    - Mocking이 되지않았기때문에 null이 return 되고, null에 대한 id를 받아왔기때문에 nullpointerexception 발생

#### 해결방법
- 1. save() 메서드로 전달되는 Event객체가 어떤 객체던간에 TEST Code에서 만든 Event객체를 생성하도록 변경
    - Mockito.any()
        - method parameter에 대한 Mocking
- 문제점
    - 입력값을 제한하였지만 입력값이 제한되어 전달됬는지에 대한 테스트가 되지않는다.
    - 다른 해결방법이 필요하다.
```java
@Test
public void 이벤트생성_테스트 () throws Exception {
    Event event = Event.builder()
            .name("Spring")
            .description("REST API Study")
            .beginEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
            .closeEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
            .beginEventDateTime(LocalDateTime.of(2019, 8, 15, 14, 21))
            .endEventDateTime(LocalDateTime.of(2019, 8, 16, 14, 21))
            .basePrice(100)
            .maxPrice(200)
            .limitOfEnrollment(100)
            .location("대전 둔산동 스타벅스")
            .free(true)
            .offline(true)
            .eventStatus(EventStatus.PUBLISHED)
            .build();

    event.setId(100);

    // EventRepository를 Mocking했기때문에 return값을 mocking해주어야함
    given(eventRepository.save(any(Event.class))).willReturn(event);

    String eventJsonString = objectMapper.writeValueAsString(event);

    this.mockMvc.perform(post("/api/events/")
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .accept(MediaTypes.HAL_JSON_UTF8)
                    .content(eventJsonString)
                )
                .andDo(print())
                .andExpect(jsonPath("$.id").value(100))
                .andExpect(status().isCreated()) // 201 응답
                .andExpect(header().exists(HttpHeaders.LOCATION))
                .andExpect(header().string(HttpHeaders.CONTENT_TYPE, MediaTypes.HAL_JSON_UTF8_VALUE))
                .andExpect(jsonPath("$.id").value(Matchers.not(1000))) // 입력값이 들어와선 안된다.
                .andExpect(jsonPath("$.free").value(Matchers.not(true))) // 입력값이 true가 나와선안됨
                .andExpect(jsonPath("$.eventStatus").value(EventStatus.DRFAT.name()));
}
```

- 2. eventRepository 를 Mocking하지않는다.
    - @SpringBootTest 로 변경
        - @WebMvcTest 는 Web과 관련된 빈들만 등록되기 때문에 EventRepository는 Bean으로 등록되지않는다.
        - EventRepository 를 주입받을수 없다.
        - WebEnvironment 속성의 기본값이 MOCK 이기 때문에 mocking된 Dispatcher Servlet을 만들도록 되어있다.
        - MockMvc 를 사용하여 테스트가 가능하다.
    - @AutoConfigureMockMvc
        - @WebMvcTest는 MockMvc설정을 자동으로 해주지만 @SpringBootTest는 그렇지않다.
        - MockMvc 를 자동으로 등록해주는 @AutoConfigureMockMvc 를 사용한다.
    - Web과 관련된 테스트시 @SpringBootTest를 사용하자
        - @WebMvcTest 를 사용하면 Mocking해야할 객체가 너무 많다.

- @SpringBootTest
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@BootstrapWith(SpringBootTestContextBootstrapper.class)
@ExtendWith({SpringExtension.class})
public @interface SpringBootTest {
    @AliasFor("properties")
    String[] value() default {};

    @AliasFor("value")
    String[] properties() default {};

    Class<?>[] classes() default {};

    SpringBootTest.WebEnvironment webEnvironment() default SpringBootTest.WebEnvironment.MOCK;

    public static enum WebEnvironment {
        MOCK(false),
        RANDOM_PORT(true),
        DEFINED_PORT(true),
        NONE(false);

        private final boolean embedded;

        private WebEnvironment(boolean embedded) {
            this.embedded = embedded;
        }

        public boolean isEmbedded() {
            return this.embedded;
        }
    }
}
```

- @AutoConfigureMockMvc
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@ImportAutoConfiguration
@PropertyMapping("spring.test.mockmvc")
public @interface AutoConfigureMockMvc {
    boolean addFilters() default true;

    @PropertyMapping(
        skip = SkipPropertyMapping.ON_DEFAULT_VALUE
    )
    MockMvcPrint print() default MockMvcPrint.DEFAULT;

    boolean printOnlyOnFailure() default true;

    @PropertyMapping("webclient.enabled")
    boolean webClientEnabled() default true;

    @PropertyMapping("webdriver.enabled")
    boolean webDriverEnabled() default true;

    /** @deprecated */
    @Deprecated
    boolean secure() default true;
}
```

- 테스트 코드
    - 입력값을 제한하여 받기때문에 free, offline, eventStatus 이 입력된 값이 등록이 되지 않아야한다.
```java
@RunWith(SpringRunner.class)
//@WebMvcTest
@SpringBootTest
@AutoConfigureMockMvc
public class EventControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Autowired
    ObjectMapper objectMapper;

//    @MockBean
    @Autowired
    EventRepository eventRepository;

    @Test
    public void 이벤트생성_테스트 () throws Exception {
        Event event = Event.builder()
                .name("Spring")
                .description("REST API Study")
                .beginEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
                .closeEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
                .beginEventDateTime(LocalDateTime.of(2019, 8, 15, 14, 21))
                .endEventDateTime(LocalDateTime.of(2019, 8, 16, 14, 21))
                .basePrice(100)
                .maxPrice(200)
                .limitOfEnrollment(100)
                .location("대전 둔산동 스타벅스")
                .free(true)
                .offline(true)
                .eventStatus(EventStatus.PUBLISHED)
                .build();

        event.setId(100);

        // EventRepository를 Mocking했기때문에 return값을 mocking해주어야함
//        given(eventRepository.save(any(Event.class))).willReturn(event);

        String eventJsonString = objectMapper.writeValueAsString(event);

        this.mockMvc.perform(post("/api/events/")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .accept(MediaTypes.HAL_JSON_UTF8)
                        .content(eventJsonString)
                    )
                    .andDo(print())
                    .andExpect(jsonPath("$.id").value(1))
                    .andExpect(status().isCreated()) // 201 응답
                    .andExpect(header().exists(HttpHeaders.LOCATION))
                    .andExpect(header().string(HttpHeaders.CONTENT_TYPE, MediaTypes.HAL_JSON_UTF8_VALUE))
                    .andExpect(jsonPath("$.id").value(Matchers.not(100))) // 입력값이 나와선안됨
                    .andExpect(jsonPath("$.free").value(Matchers.not(true))) // 입력값이 true가 나와선안됨
                    .andExpect(jsonPath("$.offline").value(Matchers.not(true))) // 입력값이 true가 나와선안됨
                    .andExpect(jsonPath("$.eventStatus").value(EventStatus.DRAFT.name())); // 입력값이 나와선안됨
    }
}
```

#### 정리
- 생성 API에 등록 요청시 원치않은 입력값이 들어올 경우 선택방안 ?
    - 1. 입력값을 제한한다.
    - 2. 원치않은 값들은 무시한다.
- 입력값을 제한하는데 사용하는 방법 ?
    - 1. DTO를 생성한다.
    - 2. 애노테이션 기반의 매핑을 사용한다.
- 모두 각각의 장단점이 존재한다.
- 필자는 좀더 확실하고 버그가 발생하지않는 방법을 선호
    - DTO를 활용하여 입력값을 제한하는 방법을 선택
    - DTO 클래스만 보아도 등록시 어떤 값들을 허용하는지 확실하게 판단가능
    - 도메인 클래스는 최대한 더럽히지 말것.
