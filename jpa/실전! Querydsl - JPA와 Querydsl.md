# 실전! Querydsl - JPA와 Querydsl

- 순수 JPA 리포지토리와 Querydsl
- 동적쿼리 Builder사용
- 동적쿼리 Where 적용
- 조회 API 컨트롤러 개발

#### 순수 JPA 리포지토리와 Querydsl
```java
@Repository
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJpaRepository(EntityManager em) {
        this.em = em;
        // 빈으로 등록해서 사용해도 되고, 생성자 내부에서 새롭게 생성해주는 방식을 사용해도 됨
        this.queryFactory = new JPAQueryFactory(em);
    }

    public void save(Member member) {
        em.persist(member);
    }

    public Optional<Member> findById(Long id) {
        Member findMember = em.find(Member.class, id);
        return Optional.ofNullable(findMember);
    }

    /*
        런타임에 쿼리가 실행될때 오류가 발생한다.
    * */
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findAll_Querydsl() {
        return queryFactory
                .selectFrom(member)
                .fetch();
    }

    public List<Member> findByUsername(String username) {
        return em.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", username)
                .getResultList();
    }

    public List<Member> findByUsername_Querydsl(String username) {
        return queryFactory
                .selectFrom(member)
                .where(member.username.eq(username))
                .fetch();
    }
}
```

`동시성 문제`
- JPAQueryFactory의 동시성 문제는 EntityManager에 의존하게 된다.
- Spring에서는 EntityManager 자체가 Thread-Safe 한 구조이다.
    - EntityManager는 트랜잭션 단위로 따로 동작하게 된다.
    - 실제 영속성 컨텍스트가 아닌 프록시 객체이고, 실제 엔티티 매니저가 트랜잭션 단위로 돌게끔 라우팅해주는 역할만 한다.
