# 실전! 스프링 데이터 JPA - 순수 JPA 기반 리포지토리 만들기

### 공통 인터페이스 기능
- 순수 JPA 기반 리포지토리 만들기
- 스프링 데이터 JPA 인터페이스 소개
- 스프링 데이터 JPA 인터페이스 활용

> 순수 JPA 기반 리포지토리로 먼저 구현을 해보고 Spring data JPA 로 변경하여 활용해보기 

#### 순수 JPA 기반 리포지토리 만들기
- 기본 CRUD
    - 저장
    - 변경 -> 변경감지
    - 삭제
    - 카운트

`MemberJpaRepository`
```java
@Repository
public class MemberJpaRepository {

    @PersistenceContext
    private EntityManager em;

    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    public void delete(Member member) {
        em.remove(member);
    }

    /**
     * 전체조회나 where 문을 활용한 조회는 JPQL을 활용하여 조회해야한다.
     * -> JPQL은 테이블을 대상을 하는것이 아닌 엔티티 객체를 대상으로 하는것이다.
     */
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    public long count() {
        return em.createQuery("select count(m) from Member m", Long.class)
                .getSingleResult();
    }

    public Member find(Long memberId) {
        return em.find(Member.class, memberId);
    }
}
```

`TeamJpaRepository`
```java
@Repository
public class TeamRepository {

    @PersistenceContext
    private EntityManager em;

    public Team save(Team team) {
        em.persist(team);
        return team;
    }

    public void delete(Team team) {
        em.remove(team);
    }

    public List<Team> findAll() {
        return em.createQuery("select t from Team t", Team.class)
                .getResultList();
    }

    public Optional<Team> findById(Long id) {
        Team team = em.find(Team.class, id);
        return Optional.ofNullable(team);
    }

    public long count() {
        return em.createQuery("select count(t) from Team t", Long.class)
                .getSingleResult();
    }
}
```

> 기본적인 등록, 수정 삭제는 타입만 다를뿐 동일하다.

#### JpaRepositoryTest
```java
// Junit 5 부터는 @RunWith 애노테이션이 더이상 필요없다.
@SpringBootTest
@Transactional
@Rollback(false)
class MemberJpaRepositoryTest {

    @Autowired MemberJpaRepository memberJpaRepository;

    /**
     * No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call 예외 발생
     * JPA의 모든 데이터조작은 트랜잭션 내에서 이루어 져야함
     */
    @Test
    public void testMember() {
        // JPA 특성상 동일 트랜잭션 내에서는 같은 식별자를 가진 엔티티는 동일성을 보장한다.
        Member member = new Member("memberA");
        Member savedMember = memberJpaRepository.save(member);

        Member findMember = memberJpaRepository.find(savedMember.getId());

        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        assertThat(findMember).isEqualTo(member);
    }

    @Test
    public void basicCRUD() {
        Member member1 = new Member("member1");
        Member member2 = new Member("member2");

        memberJpaRepository.save(member1);
        memberJpaRepository.save(member2);

        // 단건 조회 검증
        Member findMember1 = memberJpaRepository.findById(member1.getId()).get();
        Member findMember2 = memberJpaRepository.findById(member2.getId()).get();

        assertThat(findMember1).isEqualTo(member1);
        assertThat(findMember2).isEqualTo(member2);

        // 엔티티의 수정은 JPQL을 사용하는것이 아닌 변경감지를 활용하는것이 베스트 프렉티스이다.
        findMember1.setUsername("dirty checking");

        // 리스트 조회 검증
        List<Member> all = memberJpaRepository.findAll();
        assertThat(all.size()).isEqualTo(2);

        // 카운트 검증
        long count = memberJpaRepository.count();
        assertThat(count).isEqualTo(2);

        // 삭제 검증
        memberJpaRepository.delete(member1);
        memberJpaRepository.delete(member2);

        count = memberJpaRepository.count();
        assertThat(count).isEqualTo(0);
    }
}
```

> 기본적인 CRUD는 멤버, 팀 모두 동일
