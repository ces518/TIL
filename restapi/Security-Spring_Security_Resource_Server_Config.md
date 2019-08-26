# REST API - Security - Spring Security - OAuth2 Resource Server Config
- OAuth2 리소스 서버 설정
- AuthorizationServer와 연동하여 사용이 된다.
- Resource 에 접근할때 인증이 필요하다면 AuthorizationServer를 통해 token이 유효한지 확인하는 과정이 진행됨.
- ResourceServer는 API 서버와 같이 있는것이 맞음
- AuthorizationServer는 분리하는 것이 맞음

#### ResourceServer Config
- @EnableResourceServer 리소스 서버 활성화
- 익명사용자를 허용한다.
- GET /api/** 의 모든 요청들은 접근을 허용하도록 한다.
- 나머지 요청에 대해서는 인증을 진행한다.
- 인증이 실패하거나, 권한이 없는경우 예외가 발생하며 예외 발생시 OAuth2AccessDeniedHandler 를 사용한다.

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    /**
     * Resource ID 설정
     * @param resources
     * @throws Exception
     */
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId("event");
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
            .anonymous()
                .and()
            .authorizeRequests()
                .mvcMatchers(HttpMethod.GET, "/api/**").anonymous()
                .anyRequest().authenticated()
                .and()
            .exceptionHandling()
                .accessDeniedHandler(new OAuth2AccessDeniedHandler())
            // 인증이 안되거나, 권한이없는경우 예외가 발생하며 OAuth2AccessDeniedHandler 가 403 응답을 내보낸다.
        ;
    }
}
```

#### OAuth2AccessDeniedHandler
- ResourceServer에서 예외발생시 사용하도록 제공하는 ExceptionHandler
- 인증이 실패하거나, 권한이 없는경우 403 응답을 내보내준다.

```java
/**
 * If authorization fails and the caller has asked for a specific content type response, this entry point can send one,
 * along with a standard 403 status. Add to the Spring Security configuration as an {@link AccessDeniedHandler} in
 * the usual way.
 * 
 * @author Dave Syer
 * 
 */
public class OAuth2AccessDeniedHandler extends AbstractOAuth2SecurityExceptionHandler implements AccessDeniedHandler {

	public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException authException)
			throws IOException, ServletException {
		doHandle(request, response, authException);
	}

}
```

#### 기존 테스트코드 수정
- 기존의 테스트는 GET 요청을 제외하고 모두 실패한다.
- GET 을 제외한 모든 테스트는 엑세스 토큰을 가지고 요청 하도록 테스트를 수정한다.

```java
public class EventControllerTest extends BaseControllerTest {

    @Autowired
    EventRepository eventRepository;

    @Autowired
    AccountService accountService;

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
                        .header(HttpHeaders.AUTHORIZATION, getBearerToken())
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
                                    linkWithRel("update-event").description("link to update event"),
                                    linkWithRel("profile").description("link to profile")
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
                                    fieldWithPath("eventStatus").description("event status"),
                                    fieldWithPath("_links.self.href").description("link to self"),
                                    fieldWithPath("_links.query-events.href").description("link to query-events"),
                                    fieldWithPath("_links.update-event.href").description("link to update-event"),
                                    fieldWithPath("_links.profile.href").description("link to profile")
                            )
                    ))
        ;
    }

    @Test
    public void 이벤트생성_입력값이_없을경우_BAD_REQUEST () throws Exception {
        // 입력값을 아무것도 보내지않을 경우 테스트
        EventDto eventDto = EventDto.builder()
                .build();
        this.mockMvc.perform(post("/api/events")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .accept(MediaTypes.HAL_JSON_UTF8)
                        .content(objectMapper.writeValueAsString(eventDto))
                        .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                    )
                    .andDo(print())
                    .andExpect(status().isBadRequest())
                    .andExpect(jsonPath("$._links.index").exists())
        ;
    }

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
                    .content(objectMapper.writeValueAsString(eventDto))
                    .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                )
                .andDo(print())
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.content[0].objectName").exists())
                .andExpect(jsonPath("$.content[0].field").exists())
                .andExpect(jsonPath("$.content[0].defaultMessage").exists())
                .andExpect(jsonPath("$.content[0].code").exists())
                .andExpect(jsonPath("$.content[0].rejectedValue").exists())
                .andExpect(jsonPath("$._links.index").exists())
        ;
    }

    @Test
    @TestDescription("이벤트 30개를 10개씩 2번 페이지 조회하기")
    public void eventsOfSecondPage () throws Exception {
        // Given
        // 이벤트 30개 핖요
        IntStream.range(0, 30). forEach(this::generateEvent);

        // when
        this.mockMvc.perform(get("/api/events")
                    .param("page", "1") // 0부터 시작
                    .param("size", "10")
                    .param("sort", "name,DESC")
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .accept(MediaTypes.HAL_JSON_UTF8)
                )
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.page").exists())
                .andExpect(jsonPath(("$._embedded.eventList[0]._links.self")).exists())
                .andExpect(jsonPath(("$._links.self")).exists())
                .andExpect(jsonPath(("$._links.profile")).exists())
                .andDo(document("query-events",
                    links(
                            linkWithRel("first").description("첫 페이지"),
                            linkWithRel("prev").description("이전 페이지"),
                            linkWithRel("self").description("현재 페이지"),
                            linkWithRel("next").description("다음 페이지"),
                            linkWithRel("last").description("마지막 페이지"),
                            linkWithRel("profile").description("profile")
                    ),
                    requestHeaders(
                            headerWithName(HttpHeaders.ACCEPT).description("Accept Header"),
                            headerWithName(HttpHeaders.CONTENT_TYPE).description("Content Type Header")
                    ),
                    requestParameters(
                            parameterWithName("page").description("페이지 번호이며 0 부터 시작한다."),
                            parameterWithName("size").description("페이지의 사이즈"),
                            parameterWithName("sort").description("정렬 전략을 의미한다. fieldName,ASC||DESC")
                    )
                ))
        ;

        // then
    }

    private Event generateEvent(int i) {
        Event event = Event.builder()
                .name("event" + i)
                .description("test" + i)
                .beginEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
                .closeEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
                .beginEventDateTime(LocalDateTime.of(2019, 8, 15, 14, 21))
                .endEventDateTime(LocalDateTime.of(2019, 8, 16, 14, 21))
                .basePrice(100)
                .maxPrice(200)
                .limitOfEnrollment(100)
                .location("대전 둔산동 스타벅스")
                .eventStatus(EventStatus.DRAFT)
                .build();
        return this.eventRepository.save(event);
    }

    @Test
    @TestDescription("기존의 이벤트를 하나 조회")
    public void getEvent () throws Exception {
        // given
        Event event = this.generateEvent(100);
        // when & then
        this.mockMvc.perform(get("/api/events/{id}", event.getId()))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").exists())
                .andExpect(jsonPath("$.id").exists())
                .andExpect(jsonPath("$._links.self").exists())
                .andExpect(jsonPath("$._links.profile").exists())
        ;
    }

    @Test
    @TestDescription("존재하지 않는 이벤트를 조회했을때 404 응답하기")
    public void getEvent404 () throws Exception {
        // When & Then
        this.mockMvc.perform(get("/api/events/4124124"))
                .andExpect(status().isNotFound());
    }

    @Test
    @TestDescription("이벤트 정상적인 수정")
    public void updateEvent () throws Exception {
        // given
        Event event = generateEvent(200);
        EventDto eventDto = this.objectMapper.convertValue(event, EventDto.class);
        String eventName = "Update Event";
        eventDto.setName(eventName);

        // when , then
        this.mockMvc.perform(put("/api/events/{id}", event.getId())
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(eventDto))
                    .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                )
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value(eventName))
                .andExpect(jsonPath("$._links.self").exists())
        ;
    }

    @Test
    @TestDescription("입력값이 비어있는 경우 400 응답")
    public void updateEventEmptyRequestBadRequest () throws Exception {
        // given
        Event event = generateEvent(200);
        EventDto eventDto = new EventDto();

        // when , then
        this.mockMvc.perform(put("/api/events/{id}", event.getId())
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(eventDto))
                        .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                )
                .andDo(print())
                .andExpect(status().isBadRequest())
        ;
    }

    @Test
    @TestDescription("입력값이 잘못된 경우 400 응답")
    public void updateEventBadRequest () throws Exception {
        // given
        Event event = generateEvent(200);
        EventDto eventDto = this.objectMapper.convertValue(event, EventDto.class);
        eventDto.setBasePrice(20000);
        eventDto.setBasePrice(1000);

        // when , then
        this.mockMvc.perform(put("/api/events/{id}", event.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(eventDto))
                .header(HttpHeaders.AUTHORIZATION, getBearerToken())
        )
                .andDo(print())
                .andExpect(status().isBadRequest())
        ;
    }

    @Test
    @TestDescription("존재하지 않는 이벤트 수정시 404 응답")
    public void updateEventNotFound () throws Exception {
        Event event = generateEvent(200);
        EventDto eventDto = this.objectMapper.convertValue(event, EventDto.class);

        // when , then
        this.mockMvc.perform(put("/api/events/50000")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(eventDto))
                        .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                )
                .andDo(print())
                .andExpect(status().isNotFound())
        ;
    }

    private String getBearerToken () throws Exception {
        return "Bearer " + getToken();
    }

    private String getToken () throws Exception {
        final String USER_NAME = "puppee9@gmail.com";
        final String PASSWORD = "june";
        // httpBasic 메서드를 사용하여 basicOauth 헤더를 만듬
        final String CLIENT_ID = "myApp";
        final String CLIENT_SECRET = "pass";

        // given
        Set roles = new HashSet();
        roles.add(AccountRole.ADMIN);
        roles.add(AccountRole.USER);
        Account account = Account.builder()
                .email(USER_NAME)
                .password(PASSWORD)
                .roles(roles)
                .build();
        accountService.saveAccount(account);

        ResultActions perform = this.mockMvc.perform(post("/oauth/token")
                .with(httpBasic(CLIENT_ID, CLIENT_SECRET)) // httpBasic 사용시 test dependency 필요
                .param("username", USER_NAME)
                .param("password", PASSWORD)
                .param("grant_type", "password")
        );
        MockHttpServletResponse response = perform.andReturn().getResponse();
        Jackson2JsonParser parser = new Jackson2JsonParser();
        String contentAsString = response.getContentAsString();
        Map<String, Object> parseMap = parser.parseMap(contentAsString);
        return parseMap.get("access_token").toString();
    }
}
```

#### 테스트 결과 및 문제점
```java
MockHttpServletResponse:
           Status = 401
    Error message = null
          Headers = [Pragma:"no-cache", WWW-Authenticate:"Bearer error="unauthorized", error_description="query did not return a unique result: 2; nested exception is javax.persistence.NonUniqueResultException: query did not return a unique result: 2"", Cache-Control:"no-store", Content-Type:"application/json;charset=UTF-8", X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", X-Frame-Options:"DENY"]
     Content type = application/json;charset=UTF-8
             Body = {"error":"unauthorized","error_description":"query did not return a unique result: 2; nested exception is javax.persistence.NonUniqueResultException: query did not return a unique result: 2"}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
- 대다수의 테스트가 깨진다.
- 테스트가 깨지는 이유 ?
    - 매번 BearerToken() 을 가져올때 동일한 email의 유저를 생성하기때문에 해당 유저 조회시 단건의 조회가 일어나지 않음.
    - Spring Test의 특성
    - @SpringBootTest를 사용하여 테스트를 진행하기 때문에 TEST용 ApplicationContext가 생성된다.
    - InMemory DB 를 사용하기 때문에 ApplicationContext 가 죽지 않는 이상 InMemoryDB의 데이터가 공유된다.
    - InMemory DB 간의 데이터가 공유되기 때문에 이런 문제가 발생하는것

#### 해결방법
- 매 테스트 실행전 메모리 DB의 값을 초기화 해준다.
```java
@Before
public void setUp () {
    this.eventRepository.deleteAll();
    this.accountRepository.deleteAll();
}
```

#### 최종 테스트 코드 및 결과
- 모든 테스트가 성공하는것을 확인할 수 있다.
```java
public class EventControllerTest extends BaseControllerTest {

    @Autowired
    EventRepository eventRepository;

    @Autowired
    AccountService accountService;

    @Autowired
    AccountRepository accountRepository;

    @Before
    public void setUp () {
        this.eventRepository.deleteAll();
        this.accountRepository.deleteAll();
    }



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
                        .header(HttpHeaders.AUTHORIZATION, getBearerToken())
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
                                    linkWithRel("update-event").description("link to update event"),
                                    linkWithRel("profile").description("link to profile")
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
                                    fieldWithPath("eventStatus").description("event status"),
                                    fieldWithPath("_links.self.href").description("link to self"),
                                    fieldWithPath("_links.query-events.href").description("link to query-events"),
                                    fieldWithPath("_links.update-event.href").description("link to update-event"),
                                    fieldWithPath("_links.profile.href").description("link to profile")
                            )
                    ))
        ;
    }

    @Test
    public void 이벤트생성_입력값이_없을경우_BAD_REQUEST () throws Exception {
        // 입력값을 아무것도 보내지않을 경우 테스트
        EventDto eventDto = EventDto.builder()
                .build();
        this.mockMvc.perform(post("/api/events")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .accept(MediaTypes.HAL_JSON_UTF8)
                        .content(objectMapper.writeValueAsString(eventDto))
                        .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                    )
                    .andDo(print())
                    .andExpect(status().isBadRequest())
                    .andExpect(jsonPath("$._links.index").exists())
        ;
    }

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
                    .content(objectMapper.writeValueAsString(eventDto))
                    .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                )
                .andDo(print())
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.content[0].objectName").exists())
                .andExpect(jsonPath("$.content[0].field").exists())
                .andExpect(jsonPath("$.content[0].defaultMessage").exists())
                .andExpect(jsonPath("$.content[0].code").exists())
                .andExpect(jsonPath("$.content[0].rejectedValue").exists())
                .andExpect(jsonPath("$._links.index").exists())
        ;
    }

    @Test
    @TestDescription("이벤트 30개를 10개씩 2번 페이지 조회하기")
    public void eventsOfSecondPage () throws Exception {
        // Given
        // 이벤트 30개 핖요
        IntStream.range(0, 30). forEach(this::generateEvent);

        // when
        this.mockMvc.perform(get("/api/events")
                    .param("page", "1") // 0부터 시작
                    .param("size", "10")
                    .param("sort", "name,DESC")
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .accept(MediaTypes.HAL_JSON_UTF8)
                )
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.page").exists())
                .andExpect(jsonPath(("$._embedded.eventList[0]._links.self")).exists())
                .andExpect(jsonPath(("$._links.self")).exists())
                .andExpect(jsonPath(("$._links.profile")).exists())
                .andDo(document("query-events",
                    links(
                            linkWithRel("first").description("첫 페이지"),
                            linkWithRel("prev").description("이전 페이지"),
                            linkWithRel("self").description("현재 페이지"),
                            linkWithRel("next").description("다음 페이지"),
                            linkWithRel("last").description("마지막 페이지"),
                            linkWithRel("profile").description("profile")
                    ),
                    requestHeaders(
                            headerWithName(HttpHeaders.ACCEPT).description("Accept Header"),
                            headerWithName(HttpHeaders.CONTENT_TYPE).description("Content Type Header")
                    ),
                    requestParameters(
                            parameterWithName("page").description("페이지 번호이며 0 부터 시작한다."),
                            parameterWithName("size").description("페이지의 사이즈"),
                            parameterWithName("sort").description("정렬 전략을 의미한다. fieldName,ASC||DESC")
                    )
                ))
        ;

        // then
    }

    private Event generateEvent(int i) {
        Event event = Event.builder()
                .name("event" + i)
                .description("test" + i)
                .beginEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
                .closeEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
                .beginEventDateTime(LocalDateTime.of(2019, 8, 15, 14, 21))
                .endEventDateTime(LocalDateTime.of(2019, 8, 16, 14, 21))
                .basePrice(100)
                .maxPrice(200)
                .limitOfEnrollment(100)
                .location("대전 둔산동 스타벅스")
                .eventStatus(EventStatus.DRAFT)
                .build();
        return this.eventRepository.save(event);
    }

    @Test
    @TestDescription("기존의 이벤트를 하나 조회")
    public void getEvent () throws Exception {
        // given
        Event event = this.generateEvent(100);
        // when & then
        this.mockMvc.perform(get("/api/events/{id}", event.getId()))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").exists())
                .andExpect(jsonPath("$.id").exists())
                .andExpect(jsonPath("$._links.self").exists())
                .andExpect(jsonPath("$._links.profile").exists())
        ;
    }

    @Test
    @TestDescription("존재하지 않는 이벤트를 조회했을때 404 응답하기")
    public void getEvent404 () throws Exception {
        // When & Then
        this.mockMvc.perform(get("/api/events/4124124"))
                .andExpect(status().isNotFound());
    }

    @Test
    @TestDescription("이벤트 정상적인 수정")
    public void updateEvent () throws Exception {
        // given
        Event event = generateEvent(200);
        EventDto eventDto = this.objectMapper.convertValue(event, EventDto.class);
        String eventName = "Update Event";
        eventDto.setName(eventName);

        // when , then
        this.mockMvc.perform(put("/api/events/{id}", event.getId())
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(eventDto))
                    .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                )
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value(eventName))
                .andExpect(jsonPath("$._links.self").exists())
        ;
    }

    @Test
    @TestDescription("입력값이 비어있는 경우 400 응답")
    public void updateEventEmptyRequestBadRequest () throws Exception {
        // given
        Event event = generateEvent(200);
        EventDto eventDto = new EventDto();

        // when , then
        this.mockMvc.perform(put("/api/events/{id}", event.getId())
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(eventDto))
                        .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                )
                .andDo(print())
                .andExpect(status().isBadRequest())
        ;
    }

    @Test
    @TestDescription("입력값이 잘못된 경우 400 응답")
    public void updateEventBadRequest () throws Exception {
        // given
        Event event = generateEvent(200);
        EventDto eventDto = this.objectMapper.convertValue(event, EventDto.class);
        eventDto.setBasePrice(20000);
        eventDto.setBasePrice(1000);

        // when , then
        this.mockMvc.perform(put("/api/events/{id}", event.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(eventDto))
                .header(HttpHeaders.AUTHORIZATION, getBearerToken())
        )
                .andDo(print())
                .andExpect(status().isBadRequest())
        ;
    }

    @Test
    @TestDescription("존재하지 않는 이벤트 수정시 404 응답")
    public void updateEventNotFound () throws Exception {
        Event event = generateEvent(200);
        EventDto eventDto = this.objectMapper.convertValue(event, EventDto.class);

        // when , then
        this.mockMvc.perform(put("/api/events/50000")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(eventDto))
                        .header(HttpHeaders.AUTHORIZATION, getBearerToken())
                )
                .andDo(print())
                .andExpect(status().isNotFound())
        ;
    }

    private String getBearerToken () throws Exception {
        return "Bearer " + getToken();
    }

    private String getToken () throws Exception {
        final String USER_NAME = "puppee9@gmail.com";
        final String PASSWORD = "june";
        // httpBasic 메서드를 사용하여 basicOauth 헤더를 만듬
        final String CLIENT_ID = "myApp";
        final String CLIENT_SECRET = "pass";

        // given
        Set roles = new HashSet();
        roles.add(AccountRole.ADMIN);
        roles.add(AccountRole.USER);
        Account account = Account.builder()
                .email(USER_NAME)
                .password(PASSWORD)
                .roles(roles)
                .build();
        accountService.saveAccount(account);

        ResultActions perform = this.mockMvc.perform(post("/oauth/token")
                .with(httpBasic(CLIENT_ID, CLIENT_SECRET)) // httpBasic 사용시 test dependency 필요
                .param("username", USER_NAME)
                .param("password", PASSWORD)
                .param("grant_type", "password")
        );
        MockHttpServletResponse response = perform.andReturn().getResponse();
        Jackson2JsonParser parser = new Jackson2JsonParser();
        String contentAsString = response.getContentAsString();
        Map<String, Object> parseMap = parser.parseMap(contentAsString);
        return parseMap.get("access_token").toString();
    }
}
```
