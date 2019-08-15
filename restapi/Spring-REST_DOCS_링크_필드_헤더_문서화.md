# REST API - Spring - REST DOCS 링크_필드_헤더 문서화
- 요청 필드와, 헤더정보, 응답의 필드와 링크정보 에 대한 문서화가 필요하다.
- 필요한 링크정보
    - self
    - query-events
    - update-events
    - profile


#### 문서화 테스트 코드
- links(): 링크에 대한 문서화
    - linkWithRel(): 링크와 리소스와의 관계를 정의
    - description(): 링크에 대한 설명
- requestHeaders(): 요청 헤더에 대한 문서화
    - headerWithName(): 헤더의 이름
    - description(): 헤더에 대한 설명
- requestFields(): 요청 필드에 대한 문서화
    - fieldWithPath(): 요청 필드의 Path 명
    - description(): 필드에 대한 설명
- responseHeaders(): 응답 헤더에 대한 문서화
    - headerWithName(): 헤더의 이름
    - description(): 헤더에 대한 설명
- responseFields(): 응답 필드에 대한 문서화
    - fieldWithPath(): 요청 필드의 Path 명
    - description(): 필드에 대한 설명
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

    String eventJsonString = objectMapper.writeValueAsString(event);

    this.mockMvc.perform(post("/api/events")
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
                .andExpect(jsonPath("$.eventStatus").value(EventStatus.DRAFT.name()))
                .andExpect(jsonPath("$._links.self").exists())
                .andExpect(jsonPath("$._links.query-events").exists())
                .andExpect(jsonPath("$._links.update-event").exists())
                .andDo(document("create-event",
                        links(
                                linkWithRel("self").description("link to self"),
                                linkWithRel("query-events").description("link to query events"),
                                linkWithRel("update-event").description("link to update event")
                        ),
                        requestHeaders(
                                headerWithName(HttpHeaders.ACCEPT).description("Accept Header"),
                                headerWithName(HttpHeaders.CONTENT_TYPE).description("Content Type Header")
                        ),
//                            requestFields(
                        relaxedRequestFields(
                                fieldWithPath("name").description("name of new event"),
                                fieldWithPath("description").description("description of new event"),
                                fieldWithPath("beginEnrollmentDateTime").description("date time of begin of new event"),
                                fieldWithPath("closeEnrollmentDateTime").description("date time of close of new event"),
                                fieldWithPath("beginEventDateTime").description("date time of begin of new event"),
                                fieldWithPath("endEventDateTime").description("date time of end of new event"),
                                fieldWithPath("location").description("location of new event"),
                                fieldWithPath("basePrice").description("basePrice of new event"),
                                fieldWithPath("maxPrice").description("maxPrice of new event"),
                                fieldWithPath("limitOfEnrollment").description("limit of new event")
                        ),
                        responseHeaders(
                                headerWithName(HttpHeaders.LOCATION).description("Location Header"),
                                headerWithName(HttpHeaders.CONTENT_TYPE).description("Content Type Header")
                        ),
//                            responseFields(
                        relaxedResponseFields(
                                fieldWithPath("id").description("identifier of new event"),
                                fieldWithPath("name").description("name of new event"),
                                fieldWithPath("description").description("description of new event"),
                                fieldWithPath("beginEnrollmentDateTime").description("date time of begin of new event"),
                                fieldWithPath("closeEnrollmentDateTime").description("date time of close of new event"),
                                fieldWithPath("beginEventDateTime").description("date time of begin of new event"),
                                fieldWithPath("endEventDateTime").description("date time of end of new event"),
                                fieldWithPath("location").description("location of new event"),
                                fieldWithPath("basePrice").description("basePrice of new event"),
                                fieldWithPath("maxPrice").description("maxPrice of new event"),
                                fieldWithPath("limitOfEnrollment").description("limit of new event"),
                                fieldWithPath("free").description("it tells if this event is free or not"),
                                fieldWithPath("offline").description("it tells if this events is offline or not"),
                                fieldWithPath("eventStatus").description("event status")
                        )
                ))
    ;
}
```

#### relaxed Prefix Method
- relaxed 접두사가 붙지 않은 경우 욫어이나 응답의 모든 필드에 대한 문서화가 필요하다.
- 만약 요청 혹은 응답 필드중 하나라도 문서화가 되어 있지않은경우 org.springframework.restdocs.snippet.SnippetException 예외가 발생한다.

- responseFields() 를 사용하였을 경우
    - _links 도 응답 본문이기 때문에 링크에 대한 문서화가 되지않았다는 예외가 발생하게 된다.
    - 간단한 해결책으로 relaxed Prefix Method를 사용한다.
```java
org.springframework.restdocs.snippet.SnippetException: The following parts of the payload were not documented:
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/api/events/1"
    },
    "query-events" : {
      "href" : "http://localhost:8080/api/events"
    },
    "update-event" : {
      "href" : "http://localhost:8080/api/events/1"
    }
  }
}
```
- relaxed 접두사가 붙은 경우 요청이나 응답의 모든 필드에대한 문서화가 아닌 '일부' 에 대한 문서화를 진행한다.
- 모든 필드에 대한 문서화가 되지 않았더라도 일부에 대한 문서화 테스트 이기때문에 테스트가 성공적으로 끝난다.
- 장점
    - 문서의 일부부만 테스트가 가능하다.
- 단점
    - 정확한 문서를 생성하지 못한다.
- relaxed 보다는 모든 필드를 기술하는것을 권장

#### 문제점
- 링크정보에 대한 문서화를 진행했음에도 불구하고, reponseField에서 _links (링크정보) 도 응답필드 이기때문에 링크정보가 기술되지 않았다는 에러가 난다..
    - 이해가 잘 되지 않는 부분..
