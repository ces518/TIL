# REST API - Event 도메인 구현
- Event Domain Class
    - lombok 을 사용
    - javaBean spec에 준수하도록 기본 생성자, getter, setter를 생성
    - EqualsAndHashCode 를 사용하면 모든 필드를 기반으로 생성하기 때문에 of = "id" 를 명시하여 생성해주었다.
        - JPA 연관관계 매핑시 무한루프가 발생할 가능성이 있음(스택오버플로우 발생).
    - builder를 사용하면 기본생성자가 존재하지않기 때문에 NoArgsConstructor, AllArgsConstructor 를 추가
```java
@Getter @Setter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(of = "id")
@Builder
public class Event {
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
    private EventStatus eventStatus;
}
```

- EventStatus Class
```java
public enum EventStatus {
    DRAFT, PUBLISHED, BEGAN_ENROLLMENT;
}
```

- lombok 의 Builder 테스트 코드
```java
public class EventTest {

    @Test
    public void builder () {
        Event event = Event.builder().build();
        assertThat(event).isNotNull();
    }
}
```
