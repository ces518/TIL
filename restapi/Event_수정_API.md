# REST API - Event Update API
- 이벤트 수정 API
- 수정할 이벤트가 없는경우 404 응답
- 입력 데이터가 잘못된 경우 400 응답
- 도메인 로직으로 데이터 검증 실패시 400 응답
- 권한이 충분하지 않는경우 403 응답
- 정상적으로 수정 한 경우 응답
    - 200
    - 링크
    - 수정한 이벤트 데이터

#### 테스트 코드 작성
- 4가지 경우의 테스트 코드를 작성
    - 1. 정상적인 수정
    - 2. 입력값이 비어있는 경우 400 응답
    - 3. 입력데이터가 잘못된 경우 400 응답
    - 4. 존재하지않는 이벤트 수정요청시 404 응답
```java
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
            )
            .andDo(print())
            .andExpect(status().isNotFound())
    ;
}
```


#### 이벤트 수정 API
- PUT /api/events/{id} 요청
    - 이벤트가 존재하지 않는경우 404 응답
    - 잘못된 데이터가 넘어온경우 400 응답
    - 비지니스 로직상 맞지 않는경우 400 응답
    - 성공적으로 수정이 완료된 경우 200 응답
```java
@PutMapping("{id}")
public ResponseEntity updateEvent (@PathVariable Integer id,
                                    @Valid @RequestBody EventDto eventDto,
                                    Errors errors) throws JsonMappingException {
    // 이벤트가 존재하지 않는경우 404
    Optional<Event> optionalEvent = this.eventRepository.findById(id);
    if (!optionalEvent.isPresent()) {
        return ResponseEntity.notFound().build();
    }

    // 바인딩이 맞지않는경우 400
    if (errors.hasErrors()) {
        return badRequest(errors);
    }

    // 비지니스 로직상 맞지않은경우 400
    this.eventValidator.validate(eventDto, errors);
    if (errors.hasErrors()) {
        return badRequest(errors);
    }

    Event existEvent = optionalEvent.get();
    Event event = this.objectMapper.updateValue(existEvent, eventDto);
    Event savedEvent = this.eventRepository.save(event);
    EventResource eventResource = new EventResource(savedEvent);
    eventResource.add(new Link("/doc/index.html#resources-events-update").withRel("profile"));
    return ResponseEntity.ok(eventResource);
}
```
