# REST API - Event 생성 API 구현 - 입력값 제한 이외에 에러 발생
- 기존에는 받았던 값들을 무시하고 진행했지만, 이번에는 이외의 값들이 전달되면 에러를 발생시킴.

#### 테스트 코드
    - 입력값을 제한한 것 이외의 요청이 들어오면 BAD_REQUEST 응답을 보내는 테스트코드 작성
```java
@Test
public void 이벤트생성_테스트_이외의값_에러발생 () throws Exception {
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

    String eventJsonString = objectMapper.writeValueAsString(event);

    this.mockMvc.perform(post("/api/events/")
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .accept(MediaTypes.HAL_JSON_UTF8)
                    .content(eventJsonString)
            )
            .andDo(print())
            .andExpect(status().isBadRequest())// BAD_REQUEST 응답
            ;
}
```

#### 결과
- 현재는 테스트가 실패한다.
- 실패하는 이유 ?
    - 현재는 입력값을 제한한것 이외의 값이 들어오더라도 이를 '무시'하고 진행하기 때문
```java
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events/
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/hal+json;charset=UTF-8"]
             Body = {"id":100,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":true,"free":true,"eventStatus":"PUBLISHED"}
    Session Attrs = {}

Handler:
             Type = me.june.restapi.events.EventController
           Method = public org.springframework.http.ResponseEntity me.june.restapi.events.EventController.createEvent(me.june.restapi.events.EventDto)

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
             Body = {"id":1,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":false,"free":false,"eventStatus":"DRAFT"}
    Forwarded URL = null
   Redirected URL = http://localhost/api/events/1
          Cookies = []



java.lang.AssertionError: Status 
Expected :400
Actual   :201
```

#### 예외를 발생시키는 방법
- SpringBoot가 제공하는 properties를 활용한 ObjectMapper확장 기능을 활용하면된다.
- JSON를 Object로 변환하는 과정을 deserialization
- Object를 JSON 로 변환하는 과정을 serialization
- Handler에서 받지않거나 받을수 없는 값을 보낼경우 기본적으로 BAD_REQUEST를 응답한다.
- application.properties
```properties
spring.jackson.deserialization.fail-on-unknown-properties=true
```

#### REST_API 구현 방법 ?
- 입력값으로 다른 값을 같이 넘길경우 이를 무시 하는방법
    - 좀더 느슨한 방법
    - 개발 및 사용시에 편리하다.
    - 사용자에게 잘못된 사용 여지를 줄 수 있다.
- 입력값으로 다른 값을 같이 넘길경우 BAD_REQUEST를 응답하는 방법
    - 좀더 엄격한 방법
    - 개발시 좀 더 섬세한 처리가 필요하다
    - 사용자에게 잘못된 사용의 여지를 주지않음.
