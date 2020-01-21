# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 회원 도메인 개발

#### 회원 도메인 개발
`구현 기능`
- 회원등록
- 회원목록 조회

`순서`
- 회원 엔티티 구현
- 리포지토리 개발
- 서비스 개발
- 기능 테스트

`회원 엔티티`
```java
@Entity
@Getter @Setter
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

    @Embedded
    private Address address;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}
```

`회원 리포지토리`
```java
@Repository
public class MemberRepository {

    // @Autowired 를 사용해도 된다
    // @PersistenceContext 가 JPA의 표준
    // Spring Boot AutoConfigure 로 인해 자동 설정 되어있다.
    @PersistenceContext
    private EntityManager em;

    public Long save (Member member) {
        // 저장을 한뒤 member를 반환하지않고, id만 반호나함
        // 커맨드와 쿼리를 분리하라 원칙
        // 저장하는것은 가급적이면 사이드 이펙트를 일으키는 커맨드성이기 때문에 리턴값을 주지 말것.
        em.persist(member);
        return member.getId();
    }

    public Member find (Long id) {
        return em.find(Member.class, id);
    }

    public List<Member> findAll () {
        // 목록은 createQuery를 이용해 작성해야 한다.
        // JPQL은 SQL과 유사하지만, 차이점이 존재
        // JPQL은 테이블을 대상을 하는것이아닌 엔티티를 대상으로 한다는 점이다.
        return em.createQuery("select m from Member m ", Member.class)
                .getResultList();
    }

    public List<Member> findByName (String name) {
        // 이름으로 회원목록 조회
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
    }
}
```
