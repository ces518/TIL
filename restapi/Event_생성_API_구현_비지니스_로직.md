# REST API - Event 생성 API 구현 - 비즈니스 로직
- basePrice maxPrice 값에따라 무료 이벤트인지, location 에 따라 오프라인인지 , 온라인인지 비즈니스 로직을 적용한다.

#### Event 생성 API 테스트
- 무료 이벤트인지, 오프라인 이벤트인지 여부를 판단하는 테스트
```java
@Test
@TestDescription("정상적인 이벤트 생성 테스트")
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
                .andExpect(status().isCreated()) // 201 응답
                .andExpect(header().exists(HttpHeaders.LOCATION))
                .andExpect(header().string(HttpHeaders.CONTENT_TYPE, MediaTypes.HAL_JSON_UTF8_VALUE))
                .andExpect(jsonPath("$.id").value(Matchers.not(100))) // 입력값이 들어와선 안된다.
                .andExpect(jsonPath("$.free").value(false)) // 유료 이벤트
                .andExpect(jsonPath("$.offline").value(true)) // 오프라인
                .andExpect(jsonPath("$.eventStatus").value(EventStatus.DRAFT.name()));
}
```

#### Event 생성 API 변경
- 입력값을 받고나서 이벤트를 저장하기 이전 Event의 상태를 Update 하는데 Domain Class에 Update() Method를 정의
- 코드가 간결하기 때문에 ServiceLayer를 따로 생성하지 않았지만, ServiceLayer가 존재한다면 해당 비지니스 로직을 Service Layer에 작성하는것도 좋은 방법이다.
```java

public void update() {
    if (this.basePrice == 0 && this.maxPrice == 0) {
        this.free = Boolean.TRUE;
    } else {
        this.free = Boolean.FALSE;
    }
    if (this.location.trim().isEmpty()) {
        this.offline = Boolean.FALSE;
    } else {
        this.offline = Boolean.TRUE;
    }
}

@PostMapping
public ResponseEntity createEvent (@Valid @RequestBody EventDto eventDto, Errors errors) { // 입력값을 EventDto를 활용하여 받는다.
    eventValidator.validate(eventDto, errors);
    if (errors.hasErrors()) {
        return ResponseEntity.badRequest().body(errors);
    }

    Event event = objectMapper.convertValue(eventDto, Event.class);
    // 비즈니스 로직을 수행
    event.update();

    Event savedEvent = eventRepository.save(event);
    // created를 생성할때는 항상 uri를 제공해야한다.
    // org.springframework.hateoas.mvc.ControllerLinkBuilder 를 사용하면 손쉽게 URI를 생성할 수 있음.
    ControllerLinkBuilder linkBuilder = linkTo(EventController.class).slash(savedEvent.getId()); // 새로 생성된 Event의 ID를 기반으로 Location Header로 들어간다.
    URI uri = linkBuilder.toUri();
    return ResponseEntity.created(uri).body(savedEvent);
}
```

#### 테스트 결과
- 비지니스 로직에 따라 무료 이벤트인지, 오프라인 이벤트인지 여부가 적용되어 응답하는것을 확인가능하다.
```java
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events/
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/hal+json;charset=UTF-8"]
             Body = {"id":100,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":false,"free":false,"eventStatus":"PUBLISHED"}
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
             Body = {"id":1,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":true,"free":false,"eventStatus":"DRAFT"}
    Forwarded URL = null
   Redirected URL = http://localhost/api/events/1
          Cookies = []

```
