# REST API - Event API 개선
- CreateAPI 쪽을 보면 치명적인 문제가 존재한다.
- 현재 사용자의 정보를 Manager로 설정한 이후 문제가 발생 함

#### 문제점
- Manager 정보로 사용자의 민감한 정보까지 모두 내보내지고 있다.
```java
{"manager":{"id":3,"email":"user@gmail.com","password":"{bcrypt}$2a$10$erE31Q3WqXAwLRUC4NrUQug1ffumXabA3OZGZ8XXeqzVJOitqYqqe","roles":["USER","ADMIN"]}
```

#### 해결 방법
- 해결 방법은 두가지가 존재한다.
    - DTO 를 사용하여 ObjectMapper를사용하여 노출하고싶은 데이터만 Mapping하여 내보내는 방법
    - JsonSerializer를 사용하는 방법
        - 이전에 구현했던 ErrorSerializer 참고
- JsonSerializer를 사용하는 방법을 사용한다.

#### 특정 경우에만 JsonSerializer 사용하기
- @JsonComponent 로 등록한다면 매번 Account 타입의 데이터를 내보낼때 JsonSerializer를 사용하게 된다.
- Event 에서 Manager 라는 속성으로 사용중인 상태에서 Serialize 할때만 사용하고 싶다.
- @JsonSerialize
    - using: Serialize 할때 사용할 Serializer 지정
    - @JsonSerialize를 사용하면 Event 객체에서 Account 를 사용할때만 AccountSerializer 를 사용한다.
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
    private EventStatus eventStatus = EventStatus.DRAFT;

    /* Event's Owner */
    @ManyToOne
    @JsonSerialize(using = AccountSerializer.class)
    private Account manager;

    public void update() {
        if (this.basePrice == 0 && this.maxPrice == 0) {
            this.free = Boolean.TRUE;
        } else {
            this.free = Boolean.FALSE;
        }
        if (this.location == null || this.location.trim().isEmpty()) {
            this.offline = Boolean.FALSE;
        } else {
            this.offline = Boolean.TRUE;
        }
    }
}
```

#### 결과
- 기존의 민감한 정보를 포함한 응답과 달리 @JsonSerialize 애노테이션 설정 때문에
- AccountSerialzer 를 사용해서 민감한 정보를 제외한 ID 정보만 노출이 된다.
```java
{"manager":{"id":3}
```
