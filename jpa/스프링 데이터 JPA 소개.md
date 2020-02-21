# 스프링 데이터 JPA 소개
- spring.io/projects/spring-data-jpa

스프링 데이터 JPA는 JPA를 사용할때 지루하게 반복되는 코드를 자동화 해준다.

```java
@Repository
@RequiredArgsConstructor
public class MemberRepository {

    // @Autowired 를 사용해도 된다
    // @PersistenceContext 가 JPA의 표준
    // Spring Boot AutoConfigure 로 인해 자동 설정 되어있다.
    //@PersistenceContext
    private final EntityManager em;

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

```java
public interface MemberJpaRepository extends JpaRepository<Member, Long> {

    // select m from Member m where m.name = ?
    List<Member> findByName(String name);
}
```

스프링 데이터 JPA는 JpaRepository라는 인터페이스를 제공한다.

여기에 기본적인 CRUD 기능이 모두 제공된다.

findByName 처럼 일반화하기 힘든 기능도 메서드이름으로 정확한 JPQL 쿼리를 실행한다.

개발자는 인터페이스만 정의하면 되며, 구현체는 스프링 데이터 JPA가 애플리케이션 실행시점에 주입해 준다.

> Spring Data JPA 는 JPA를 사용해서 이런 기능을 제공할 뿐, JPA 자체를 잘 이해하는것이 중요하다.
