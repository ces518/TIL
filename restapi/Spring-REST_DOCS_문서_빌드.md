# REST API - Spring - REST DOCS 문서 빌드하기
#### Maven Plugin 추가
```xml
<plugins>
<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>1.5.3</version>
    <executions>
        <execution>
            <id>generate-docs</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>process-asciidoc</goal>
            </goals>
            <configuration>
                <backend>html</backend>
                <doctype>book</doctype>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-asciidoctor</artifactId>
            <version>${spring-restdocs.version}</version>
        </dependency>
    </dependencies>
</plugin>
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.7</version>
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <outputDirectory>
                    ${project.build.outputDirectory}/static/docs </outputDirectory>
                <resources>
                    <resource>
                        <directory>
                            ${project.build.directory}/generated-docs
                        </directory>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
</plugins>
```
#### asciidoc template 파일 생성
- src/main/asciidoc/index.adoc 생성
```adoc
= Natural REST API Guide
백기선;
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 4
:sectlinks:
:operation-curl-request-title: Example request
:operation-http-response-title: Example response

[[overview]]
= 개요

[[overview-http-verbs]]
== HTTP 동사

본 REST API에서 사용하는 HTTP 동사(verbs)는 가능한한 표준 HTTP와 REST 규약을 따릅니다.

|===
| 동사 | 용례

| `GET`
| 리소스를 가져올 때 사용

| `POST`
| 새 리소스를 만들 때 사용

| `PUT`
| 기존 리소스를 수정할 때 사용

| `PATCH`
| 기존 리소스의 일부를 수정할 때 사용

| `DELETE`
| 기존 리소스를 삭제할 떄 사용
|===

[[overview-http-status-codes]]
== HTTP 상태 코드

본 REST API에서 사용하는 HTTP 상태 코드는 가능한한 표준 HTTP와 REST 규약을 따릅니다.

|===
| 상태 코드 | 용례

| `200 OK`
| 요청을 성공적으로 처리함

| `201 Created`
| 새 리소스를 성공적으로 생성함. 응답의 `Location` 헤더에 해당 리소스의 URI가 담겨있다.

| `204 No Content`
| 기존 리소스를 성공적으로 수정함.

| `400 Bad Request`
| 잘못된 요청을 보낸 경우. 응답 본문에 더 오류에 대한 정보가 담겨있다.

| `404 Not Found`
| 요청한 리소스가 없음.
|===

[[overview-errors]]
== 오류

에러 응답이 발생했을 때 (상태 코드 >= 400), 본문에 해당 문제를 기술한 JSON 객체가 담겨있다. 에러 객체는 다음의 구조를 따른다.

include::{snippets}/errors/response-fields.adoc[]

예를 들어, 잘못된 요청으로 이벤트를 만들려고 했을 때 다음과 같은 `400 Bad Request` 응답을 받는다.

include::{snippets}/errors/http-response.adoc[]

[[overview-hypermedia]]
== 하이퍼미디어

본 REST API는 하이퍼미디어와 사용하며 응답에 담겨있는 리소스는 다른 리소스에 대한 링크를 가지고 있다.
응답은 http://stateless.co/hal_specification.html[Hypertext Application from resource to resource. Language (HAL)] 형식을 따른다.
링크는 `_links`라는 키로 제공한다. 본 API의 사용자(클라이언트)는 URI를 직접 생성하지 않아야 하며, 리소스에서 제공하는 링크를 사용해야 한다.

[[resources]]
= 리소스

[[resources-index]]
== 인덱스

인덱스는 서비스 진입점을 제공한다.


[[resources-index-access]]
=== 인덱스 조회

`GET` 요청을 사용하여 인덱스에 접근할 수 있다.

operation::index[snippets='response-body,http-response,links']

[[resources-events]]
== 이벤트

이벤트 리소스는 이벤트를 만들거나 조회할 때 사용한다.

[[resources-events-list]]
=== 이벤트 목록 조회

`GET` 요청을 사용하여 서비스의 모든 이벤트를 조회할 수 있다.

operation::get-events[snippets='response-fields,curl-request,http-response,links']

[[resources-events-create]]
=== 이벤트 생성

`POST` 요청을 사용해서 새 이벤트를 만들 수 있다.

operation::create-event[snippets='request-fields,curl-request,http-request,request-headers,http-response,response-headers,response-fields,links']

[[resources-events-get]]
=== 이벤트 조회

`Get` 요청을 사용해서 기존 이벤트 하나를 조회할 수 있다.

operation::get-event[snippets='request-fields,curl-request,http-response,links']

[[resources-events-update]]
=== 이벤트 수정

`PUT` 요청을 사용해서 기존 이벤트를 수정할 수 있다.

operation::update-event[snippets='request-fields,curl-request,http-response,links']

```

- operation::create-event[snippets='request-fields,curl-request,http-request,request-headers,http-response,response-headers,response-fields,links']
    - create-event snippets 들을 끼워 넣겠다고 선언
- 그런 다음 maven lifecycle을 활용하여 플러그인으로 build

#### 문서 생성
- maven package 실행 
    - Compile
    - Test Compile
    - 컴파일 시 RestDocs Test 때문에 문서 조각들이 생성이 됨
    - 위에 추가한 플러그인들 때문에 API 문서 페이지가 생성됨
    - target/generated-docs/index.html 파일이 생성된다.

- 지금 까지는 HATEOAS만 지켜졌지만, 생성된 문서를 통해 self-description 을 지킬수 있음.

#### 플러그인 살펴보기
- prepare-package phase
    - process-asciidoc 기능이 실행됨
    - src/main/asciidoc 하위의 모든 .adoc 파일을 문서로 만들어준다.
    - target/generated-docs/index.html 파일로 생성해준다.
    - localhost:8080/docs/index.html 로 배포가 된다.
- copy-resource
    - target/generated-docs/index.html 파일을 tatic/docs/index.html 로 생성 해 준다.
- 플러그인의 선언 순서도 중요하다
    - 동일한 phase 에 끼워넣었기때문에 순서가 중요함
    - static/doc/index.html 이 먼저 생성되고 난뒤 copy가 이루어질수 있기때문에 순서에 유의할것.
```xml
<configuration>
    <outputDirectory>
        ${project.build.outputDirectory}/static/docs </outputDirectory>
    <resources>
        <resource>
            <directory>
                ${project.build.directory}/generated-docs
            </directory>
        </resource>
    </resources>
</configuration>
```

#### 응답에 profile 링크 추가하기
- 테스트 코드 추가
    - linkWithRel("profile").description("link to profile") 링크에 profile이 존재하는지 테스트
- Controller 에서 profile Link 를 보내주지 않았기 때문에 테스트는 실패함
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
                        responseFields(
//                            relaxedResponseFields(
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
```

- Event 생성 API 변경
    - profile 링크 추가 
        - /docs/index.html 
```java
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
