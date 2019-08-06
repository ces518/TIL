# REST API - Event 생성 API 구현 - EventRepository
#### Event Entity
```java
@Getter @Setter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(of = "id")
@Builder
@Entity
public class Event {

    @Id @GeneratedValue
    private Integer id;

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

    private boolean offline;

    private boolean free;

    @Enumerated(EnumType.STRING)
    private EventStatus eventStatus;
}
```

#### Enum Mapping시 주의할점
- @Enumerated(EnumType.STRING) 을 사용할것.
- ORDINAL 을 사용하면, ENUM의 순서가 바뀌게되면 기존의 데이터에 영향을 미치게된다.

#### Spring Data JPA
    - JpaRepository를 상속받아 생성
    - jpaRepository는 상위클래스에 CRUD Repository 가 존재하기 때문에 기본적인 CRUD메서드가 존재한다.
```java
public interface EventRepository extends JpaRepository<Event, Long> {
}
```

- JpaRepository
```java
package org.springframework.data.jpa.repository;

import java.util.List;
import org.springframework.data.domain.Example;
import org.springframework.data.domain.Sort;
import org.springframework.data.repository.NoRepositoryBean;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.repository.query.QueryByExampleExecutor;

@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    List<T> findAll();

    List<T> findAll(Sort var1);

    List<T> findAllById(Iterable<ID> var1);

    <S extends T> List<S> saveAll(Iterable<S> var1);

    void flush();

    <S extends T> S saveAndFlush(S var1);

    void deleteInBatch(Iterable<T> var1);

    void deleteAllInBatch();

    T getOne(ID var1);

    <S extends T> List<S> findAll(Example<S> var1);

    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
}
```

#### Event 생성 API
    - produces = MediaTypes.HAL_JSON_UTF8_VALUE
        - 응답 컨텐츠 타입을 HAL_JSON 으로 지정한다.
    - 생성 API 로 이벤트 생성 요청이 오면, 해당 이벤트를 EventRepository에 저장한다.
    - 저장된 Event의 ID로 링크를 생성해준다.
    - 그후 저장된 Event를 응답본문으로 Return
```java
@RestController
@RequestMapping(value = "/api/events", produces = MediaTypes.HAL_JSON_UTF8_VALUE)
public class EventController {

    private final EventRepository eventRepository;

    public EventController(EventRepository eventRepository) {
        this.eventRepository = eventRepository;
    }

    @PostMapping
    public ResponseEntity createEvent (@RequestBody Event event) {

        Event savedEvent = eventRepository.save(event);
        // created를 생성할때는 항상 uri를 제공해야한다.
        // org.springframework.hateoas.mvc.ControllerLinkBuilder 를 사용하면 손쉽게 URI를 생성할 수 있음.
        ControllerLinkBuilder linkBuilder = linkTo(EventController.class).slash(savedEvent.getId()); // 새로 생성된 Event의 ID를 기반으로 Location Header로 들어간다.
        URI uri = linkBuilder.toUri();
        return ResponseEntity.created(uri).body(savedEvent);
    }
}
```
#### Event 생성 API 테스트
- @WebMvcTest
    - 웹과 관련된 테스트 이기때문에 EventRepository 를 @Autowired로 의존성을 주입받아 사용하려고하면 해당 빈이 존재하지않는다는 에러가 발생한다.
- @MockBean
    - @MockBean 애노테이션을 활용하여 해당 객체를 Mocking한다.
- given(eventRepository.save(event)).willReturn(event);
    - Mocking된 객체이기때문에 해당 객체의 특정 메서드를 호출하였을때 Return값도 지정해주어야한다.

```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class EventControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Autowired
    ObjectMapper objectMapper;

    @MockBean
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
                .build();

        event.setId(1);

        // EventRepository를 Mocking했기때문에 return값을 mocking해주어야함
        given(eventRepository.save(event)).willReturn(event);

        String eventJsonString = objectMapper.writeValueAsString(event);

        this.mockMvc.perform(post("/api/events/")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .accept(MediaTypes.HAL_JSON_UTF8)
                        .content(eventJsonString)
                    )
                    .andDo(print())
                    .andExpect(jsonPath("$.id").value("1"))
                    .andExpect(status().isCreated()) // 201 응답
                    .andExpect(header().exists(HttpHeaders.LOCATION))
                    .andExpect(header().string(HttpHeaders.CONTENT_TYPE, MediaTypes.HAL_JSON_UTF8_VALUE));
    }
}
```
