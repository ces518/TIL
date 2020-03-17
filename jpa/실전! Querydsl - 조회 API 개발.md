# 실전! Querydsl - 조회 API 개발

#### 조회 API 개발
- 편리한 데이터 확인을 위해 샘플 데이터를 추가한다.
- 샘플 데이터 추가가 테스트 케이스 실행에 영향을 주지 않도록 다음과 같이 프로파일 설정

`local`
```yaml
spring:
  profiles:
    active: local
  datasource:
    url: jdbc:h2:tcp://localhost/~/querydsl
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
#        show_sql: true
        format_sql: true
        use_sql_comments: true


logging.level:
  org.hibernate.SQL: debug
```

`test`
```yaml
spring:
  profiles:
    active: test
  datasource:
    url: jdbc:h2:tcp://localhost/~/querydsl
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
#        show_sql: true
        format_sql: true
        use_sql_comments: true


logging.level:
  org.hibernate.SQL: debug
```

> 프로파일을 활용해서 테스트 케이스에서는 샘플 데이터를 등록하는 로직이 동작 하지않도록 분리한다.


##### 샘플 데이터 작성
```java
@Profile("local")
@Component
@RequiredArgsConstructor
public class InitMember {

    private final InitMemberService initMemberService;

    @PostConstruct
    public void init() {
        initMemberService.init();
    }

    // 바로 PostConstruct에 넣는 않는 이유
    // Spring lifeCycle 이 존재해서 PostConstruct에서 @Transactional을 사용할 경우
    // AOP 트랜잭션 처리를 보장할 수 없음.
    @Component
    static class InitMemberService {
        @PersistenceContext
        private EntityManager em;

        @Transactional
        public void init() {
            Team teamA = new Team("teamA");
            Team teamB = new Team("teamB");
            em.persist(teamA);
            em.persist(teamB);

            for (int i = 0; i< 100; i++) {
                Team selectedTeam = i % 2 == 0 ? teamA : teamB;
                em.persist(new Member("member" + i, i, selectedTeam));
            }
        }
    }
}
```
- Profile이 local 일 경우에만 테스트 데이터가 동작하도록 작성
    - Test 코드에 영향이 미치면 안된다.

`PostConstruct 내에서 샘플 데이터 Persist를 하지 않는이유`
- JPA에서의 데이터 조작은 모두 트랜잭션 내에서 이루어 져야한다.
- Spring @Transactional 애노테이션을 @PostConstruct 와 같이 사용하게 될 겨웅
- lifeCycle로 인해 AOP를 통한 트랜잭션 적용이 보장되지 않는다.

##### API
`MemberController`
```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberJpaRepository memberJpaRepository;

    @GetMapping("/v1/members")
    public List<MemberTeamDto> searchMemberV1(MemberSearchCondition condition) {
        return memberJpaRepository.search(condition);
    }
}
```

`요청 결과`
- localhost:8080/v1/members?teamName=teamB&ageGoe=31&ageLoe=35
```json
[
    {
        "memberId": 34,
        "username": "member31",
        "age": 31,
        "teamId": 2,
        "teamName": "teamB"
    },
    {
        "memberId": 36,
        "username": "member33",
        "age": 33,
        "teamId": 2,
        "teamName": "teamB"
    },
    {
        "memberId": 38,
        "username": "member35",
        "age": 35,
        "teamId": 2,
        "teamName": "teamB"
    }
]
```
