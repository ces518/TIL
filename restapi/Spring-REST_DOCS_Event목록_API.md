# REST API - Spring - REST DOCS Event list API
- 이벤트 목록 조회 API 구현

#### 필요한것
- Event 목록 Paging 정보와 함께 Query
- Sort, Paging 여부 확인
- Event -> EventResource로 변환하여 받기
    - 각 EventResource 마다 Self 존재 여부 확인
- 링크 정보 확인
    - self
    - profile
- 문서화

#### 테스트 코드 작성
- 이벤트 데이터 30개 중 10개씩 2번 페이지를 조회하는 테스트 코드
- 테스트 코드를 하나하나 살펴보자.
```java
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

private void generateEvent(int i) {
    Event event = Event.builder()
            .name("event" + i)
            .description("test" + i)
            .build();
    this.eventRepository.save(event);
}
```

- 이벤트 데이터 30개 생성
    - 페이징 이전에 테스트 데이터가 필요하기때문에 30개의 테스트 데이터를 생성한다.
```java
IntStream.range(0, 30). forEach(this::generateEvent);
```

- 2페이지에 해당하는 이벤트 목록을 요청
    - GET /api/events?page=1&size=10&sort=name,DESC로 요청을 보낸다.
    - 페이징과 관련된 파라메터들은 어떻게 처리해야할까 ?
```java
this.mockMvc.perform(get("/api/events")
            .param("page", "1") // 0부터 시작
            .param("size", "10")
            .param("sort", "name,DESC")
            .contentType(MediaType.APPLICATION_JSON_UTF8)
            .accept(MediaTypes.HAL_JSON_UTF8)
        )
```

#### 페이징 과 정렬
- Spring Data Jpa 가 제공하는 Pageable
    - paging 과 관련된 정보들을 받아올 수 있음.
```java
@GetMapping
public ResponseEntity getEvents (Pageable pageable) { // paging과 관련된 파라메터들을 받아올 수 있음.
    Page<Event> pagedEvents = this.eventRepository.findAll(pageable);
    return ResponseEntity.ok(pagedEvents);
}
```

- Pageable.java
```java
package org.springframework.data.domain;

import java.util.Optional;
import org.springframework.util.Assert;

public interface Pageable {
    static Pageable unpaged() {
        return Unpaged.INSTANCE;
    }

    default boolean isPaged() {
        return true;
    }

    default boolean isUnpaged() {
        return !this.isPaged();
    }

    int getPageNumber();

    int getPageSize();

    long getOffset();

    Sort getSort();

    default Sort getSortOr(Sort sort) {
        Assert.notNull(sort, "Fallback Sort must not be null!");
        return this.getSort().isSorted() ? this.getSort() : sort;
    }

    Pageable next();

    Pageable previousOrFirst();

    Pageable first();

    boolean hasPrevious();

    default Optional<Pageable> toOptional() {
        return this.isUnpaged() ? Optional.empty() : Optional.of(this);
    }
}
```

#### 문제점
- 페이징 처리와 함께 첫페이지, 이전페이지, 현재페이지, 다음페이지, 마지막 페이지에 대한 링크 정보는 제공되지만, 각 Event의 상세보기에 해당하는 링크정보는 제공되고 있지 않다.

#### 해결 방안
- 먼저 페이징처리된 Event 목록을 Resource list로 변경해야한다.
- 그런 다음 각 Event 들을 Resource<Event> 로 변경하여 제공 해야한다.

#### PagedResourceAssembler<T>
- Page<T> 를 페이징처리가 된 Resource<T> 목록으로 변환해준다.
- 또한 각 Event를 Resource<Event> 로 변환 작업이 필요하다.
```java
@GetMapping
public ResponseEntity getEvents (Pageable pageable, PagedResourcesAssembler<Event> assembler) { // paging과 관련된 파라메터들을 받아올 수 있음.
    Page<Event> pagedEvents = this.eventRepository.findAll(pageable);
    PagedResources<Resource<Event>> pagedResources = assembler.toResource(pagedEvents, e -> new EventResource(e));
    return ResponseEntity.ok(pagedResources);
}
```

#### Profile
- 마지막으로 Profile 에 대한 링크 정보만 추가해주면 필요한 링크정보는 모두 제공하는 셈이다.
- Resource로 변환 되면 링크정보를 추가할수 있는 Method를 가지고 있기 때문에 PagedResources 에 profile 링크를 추가해준다.
```java
@GetMapping
public ResponseEntity getEvents (Pageable pageable, PagedResourcesAssembler<Event> assembler) { // paging과 관련된 파라메터들을 받아올 수 있음.
    Page<Event> pagedEvents = this.eventRepository.findAll(pageable);
    PagedResources<Resource<Event>> pagedResources = assembler.toResource(pagedEvents, e -> new EventResource(e));
    pagedResources.add(new Link("/docs/index.html#resources-events-list").withRel("profile"));
    return ResponseEntity.ok(pagedResources);
}
```

#### 최종 테스트 결과
- 테스트는 성공적으로 끝나고, target/generated-snippets/query-events 디렉터리에 snippets가 생성된것을 확인할 수 있다.
```java
MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/hal+json;charset=UTF-8"]
     Content type = application/hal+json;charset=UTF-8
             Body = {"_embedded":{"eventList":[{"id":27,"name":"event26","description":"test26","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/27"}}},{"id":26,"name":"event25","description":"test25","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/26"}}},{"id":25,"name":"event24","description":"test24","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/25"}}},{"id":24,"name":"event23","description":"test23","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/24"}}},{"id":23,"name":"event22","description":"test22","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/23"}}},{"id":22,"name":"event21","description":"test21","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/22"}}},{"id":21,"name":"event20","description":"test20","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/21"}}},{"id":3,"name":"event2","description":"test2","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/3"}}},{"id":20,"name":"event19","description":"test19","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/20"}}},{"id":19,"name":"event18","description":"test18","beginEnrollmentDateTime":null,"closeEnrollmentDateTime":null,"beginEventDateTime":null,"endEventDateTime":null,"location":null,"basePrice":0,"maxPrice":0,"limitOfEnrollment":0,"offline":false,"free":false,"eventStatus":null,"_links":{"self":{"href":"http://localhost:8080/api/events/19"}}}]},"_links":{"first":{"href":"http://localhost:8080/api/events?page=0&size=10&sort=name,desc"},"prev":{"href":"http://localhost:8080/api/events?page=0&size=10&sort=name,desc"},"self":{"href":"http://localhost:8080/api/events?page=1&size=10&sort=name,desc"},"next":{"href":"http://localhost:8080/api/events?page=2&size=10&sort=name,desc"},"last":{"href":"http://localhost:8080/api/events?page=2&size=10&sort=name,desc"},"profile":{"href":"/docs/index.html#resources-events-list"}},"page":{"size":10,"totalElements":30,"totalPages":3,"number":1}}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
