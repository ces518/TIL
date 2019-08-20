# REST API - Spring - REST DOCS Event View API
- 이벤트 상세 조회 API 구현
- 조회하는 이벤트가 존재할 경우 이벤트 리소스 확인
    - self
    - profile
    - update
- 이벤트 데이터
- 이벤트가 없을경우 404 응답

#### 이벤트 조회 API 테스트 코드
- 두가지 테스트 코드를 작성
- GET /api/events/{id} 로 요청을하면 이벤트가 존재할경우 이벤트 리소스를 제공 하는 테스트 코드,
- 존재하지 않는 이벤트를 요청했다면 404 응답을 하는 테스트 코드
```java
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
```

#### 이벤트 조회 API
- id 로 이벤트를 조회하고 Optional 을 받음.
- Event가 존재하지않으면 404 응답
- Event가 존재할경우 profile링크 정보와 함께 eventResource 제공
```java
@GetMapping("{id}")
public ResponseEntity getEvent (@PathVariable Integer id) {
    Optional<Event> optionalEvent = this.eventRepository.findById(id);
    if (!optionalEvent.isPresent()) {
        return ResponseEntity.notFound().build();
    }
    Event event = optionalEvent.get();
    EventResource eventResource = new EventResource(event);
    eventResource.add(new Link("/docs/index.html#resources-events-list").withRel("profile"));
    return ResponseEntity.ok(eventResource);
}
```
