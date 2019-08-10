# REST API - Event 생성 API 구현 - BAD_REQUEST 처리하기
- 입력값이 없다면 BAD_REQUEST 를 발생시킴.

#### 테스트 코드
- 아무런 입력값도 받지 않을경우 BAD_REQUEST를 받는 테스트코드 
```java
@Test
public void 이벤트생성_입력값이_없을경우_BAD_REQUEST () throws Exception {
    // 입력값을 아무것도 보내지않을 경우 테스트
    EventDto eventDto = EventDto.builder()
            .build();

    this.mockMvc.perform(post("/api/events")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaTypes.HAL_JSON_UTF8)
                .content(objectMapper.writeValueAsString(eventDto)))
                .andDo(print())
                .andExpect(status().isBadRequest());
}
```

#### 테스트 결과
- 400 BAD_REQUEST를 기대했지만 201 CREATED 응답이 온다.
- 잘못된 결과
```java
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/hal+json;charset=UTF-8"]
             Body = {"name":null,"description":null,"beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0}
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
           Status = 201
    Error message = null
          Headers = [Location:"http://localhost/api/events/1", Content-Type:"application/hal+json;charset=UTF-8"]
     Content type = application/hal+json;charset=UTF-8
             Body = {"id":1,"name":null,"description":null,"beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":"DRAFT"}
    Forwarded URL = null
   Redirected URL = http://localhost/api/events/1
          Cookies = []



java.lang.AssertionError: Status 
Expected :400
Actual   :201
```

#### @Valid
- @Valid를 사용하면 Handler Method에서 데이터를 바인딩시 검증을 진행한다.
- JSR 303 애노테이션을 활용해서 검증을 진행할 수 있다.
- 애노테이션들의 정보를 참고해서 검증을 수행한다.
- 검증을 수행한 결과를 Errors Type의 객체에다가 해당 검증 에러를 담아준다.

#### Handler 코드 변경
    - @Valid 를 사용하여 데아터 검증을 진행한다.
    - 입력값이 없을경우 BAD_REQUEST를 응답한다.

- EventDTO
```java
@Getter @Setter
@Builder @AllArgsConstructor @NoArgsConstructor
public class EventDto {

    @NotBlank
    private String name;

    @NotBlank
    private String description;

    @NotNull
    private LocalDateTime beginEnrollmentDateTime;

    @NotNull
    private LocalDateTime closeEnrollmentDateTime;

    @NotNull
    private LocalDateTime beginEventDateTime;

    @NotNull
    private LocalDateTime endEventDateTime;

    private String location; // optional 없다면 온라인모임

    @Min(0)
    private int basePrice;

    @Min(0)
    private int maxPrice;

    @Min(0)
    private int limitOfEnrollment;
}
```

- 이벤트 생성 API
```java
@PostMapping
public ResponseEntity createEvent (@Valid @RequestBody EventDto eventDto, Errors errors) { // 입력값을 EventDto를 활용하여 받는다.
    if (errors.hasErrors()) {
        return ResponseEntity.badRequest().build();
    }

    Event event = objectMapper.convertValue(eventDto, Event.class);

    Event savedEvent = eventRepository.save(event);
    // created를 생성할때는 항상 uri를 제공해야한다.
    // org.springframework.hateoas.mvc.ControllerLinkBuilder 를 사용하면 손쉽게 URI를 생성할 수 있음.
    ControllerLinkBuilder linkBuilder = linkTo(EventController.class).slash(savedEvent.getId()); // 새로 생성된 Event의 ID를 기반으로 Location Header로 들어간다.
    URI uri = linkBuilder.toUri();
    return ResponseEntity.created(uri).body(savedEvent);
}
```

#### 변경된 코드 테스트 결과
- 입력값이 없는경우 테스트를 진행
- 기대값인 400 BAD_REQUEST가 나온다.
```java
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/hal+json;charset=UTF-8"]
             Body = {"name":null,"description":null,"beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0}
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
          Headers = []
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

## 시작일이 종료일을 넘을경우 테스트

#### 테스트코드
```java
@Test
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
            .andExpect(status().isBadRequest());
}
```

#### 테스트 결과
- 시작일이 종료일을 넘는 경우와 같은 케이스는 애노테이션만으로 검증이 불가능하다.
- 별도의 Validator를 구현해주어야함
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
           Status = 201
    Error message = null
          Headers = [Location:"http://localhost/api/events/1", Content-Type:"application/hal+json;charset=UTF-8"]
     Content type = application/hal+json;charset=UTF-8
             Body = {"id":1,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-10-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-10-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":false,"free":false,"eventStatus":"DRAFT"}
    Forwarded URL = null
   Redirected URL = http://localhost/api/events/1
          Cookies = []



java.lang.AssertionError: Status 
Expected :400
Actual   :201
```

#### EventValidator 구현
- Validator Interface를 구현하지않고, EventValidator를 Bean으로 등록하여 이를 주입 받아 사용
```java
@Component
public class EventValidator{

    public void validate (EventDto eventDto, Errors errors) {
        // 무제한 경매가 아닌데, basePrice 가 max보다 큰 경우
        if (eventDto.getBasePrice() > eventDto.getMaxPrice() && eventDto.getMaxPrice() > 0) {
            errors.rejectValue("basePrice", "wrongValue", "BasePrice is must be less than MaxPrice");
        }

        LocalDateTime endEventDateTime = eventDto.getEndEventDateTime();
        if (endEventDateTime.isBefore(eventDto.getBeginEventDateTime()) ||
        endEventDateTime.isBefore(eventDto.getCloseEnrollmentDateTime()) ||
        endEventDateTime.isBefore(eventDto.getBeginEnrollmentDateTime())) {
            errors.rejectValue("endEventDateTime", "wrongValue", "EndEventDateTime is must be more than begin");
        }

    }
}
```

#### Event 생성 API 코드 변경
- 빈으로 등록된 EventValidator를 주입받는다.
- EventValidator를 통한 검증도 추가
```java
private final ObjectMapper objectMapper;
private final EventRepository eventRepository;
private final EventValidator eventValidator;

public EventController(ObjectMapper objectMapper, EventRepository eventRepository, EventValidator eventValidator) {
    this.objectMapper = objectMapper;
    this.eventRepository = eventRepository;
    this.eventValidator = eventValidator;
}

@PostMapping
public ResponseEntity createEvent (@Valid @RequestBody EventDto eventDto, Errors errors) { // 입력값을 EventDto를 활용하여 받는다.
    eventValidator.validate(eventDto, errors);
    if (errors.hasErrors()) {
        return ResponseEntity.badRequest().build();
    }

    Event event = objectMapper.convertValue(eventDto, Event.class);

    Event savedEvent = eventRepository.save(event);
    // created를 생성할때는 항상 uri를 제공해야한다.
    // org.springframework.hateoas.mvc.ControllerLinkBuilder 를 사용하면 손쉽게 URI를 생성할 수 있음.
    ControllerLinkBuilder linkBuilder = linkTo(EventController.class).slash(savedEvent.getId()); // 새로 생성된 Event의 ID를 기반으로 Location Header로 들어간다.
    URI uri = linkBuilder.toUri();
    return ResponseEntity.created(uri).body(savedEvent);
}
```

#### 변경된 코드 테스트 결과
- 기대값인 400 BAD_REQUEST를 응답한다.
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
          Headers = []
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
