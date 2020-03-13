# 실전! Querydsl - JPQL vs Querydsl

#### JPQL vs Querydsl
```java
@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Autowired
    EntityManager em;

//    멀티스레드에 문제없도록 설계되어 있음.
//    EntityManager가 스프링에서 스레드 세이프하게 동작한다.
    JPAQueryFactory queryFactory;

    /**
     * @BeforeEach 각 테스트 실행전에 실행되는 로직
     * 테스트 전처리 작업시 사용한다.
     */
    @BeforeEach
    public void before() {
        queryFactory = new JPAQueryFactory(em);

        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        em.persist(teamA);
        em.persist(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 20, teamA);

        Member member3 = new Member("member3", 30, teamB);
        Member member4 = new Member("member4", 40, teamB);

        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);
    }


    @Test
    public void startJPQL() {
        // member1 조회
        // 런타임에 오류 체크
        Member findMember = em.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", "member1")
                .getSingleResult();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }

    @Test
    public void startQuerydsl() {
//        JPAQueryFactory queryFactory = new JPAQueryFactory(em);
        QMember m = new QMember("m"); // QMember를 구분하는 이름 (Alias) 크게 중요하진 않음

        // JDBC의 PrepareStatement로 자동으로 파라메터 바인딩을 해준다.
        // > SQL Injection도 방지됨.
        // 컴파일 시점에 오류 체크
        Member findMember = queryFactory
                .select(m)
                .from(m)
                .where(m.username.eq("member1"))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```

`@BeforeEach`
- 각 테스트 코드 실행이전에 실행된다.
- 테스트 전처리 작업시 사용


`JPQL`
- 문자열을 이용해서 쿼리를 생성한다.
    - 타입 세이프 하지 않다.
    - 오타 (휴면에러)등을 런타임 시점에 체크가 가능하다.

`Querydsl`
- JPAQueryFactory를 이용한다.
- QType을 이용해 자바코드로 쿼리를 할 수 있다.
    - 타입 세이프하다.
    - 오타 (휴면에러)등을 **컴파일 시점**에 체크가 가능하다.
- JDBC의 PrepareStatement를 이용하여 파라메터 바인딩을 해준다.
- 직관적이다.
