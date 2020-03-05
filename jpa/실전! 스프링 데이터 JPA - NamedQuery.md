# 실전! 스프링 데이터 JPA - NamedQuery

#### JPA NamedQuery
- `@NamedQuery` 어노테이션으로 Name 쿼리 정의
- 쿼리에 이름을 정의하여 사용하는 기능

`Member`
```java
@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = { "id", "username", "age" }) // 연관관계 필드는 무한루프에 빠질수 있기때문에 ToString대상에서 제외할것
@NamedQuery(
        name = "Member.findByUsername",
        query = "select m from Member m where m.username = :username"
)
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    private String username;
    private int age;

    // 연관관계는 반드시 LAZY로 지정
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;

    public Member(String username) {
        this.username = username;
    }

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        this.team = team;
    }

    public Member(String username, int age) {
        this.username = username;
        this.age = age;
    }

    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```

`MemberJpaRepository`
```java
public List<Member> findByUsername(String username) {
    return em.createNamedQuery("Member.findByUsername", Member.class)
            .setParameter("username", username)
            .getResultList();
}
```

- 엔티티 상단에 @NamedQuery 애노테이션을 사용하여 NamedQuery를 정의하여 리포지토리에서 가져다 쓴다.

`장점`
- 애플리케이션 로딩 시점에 에러 체크가 가능하다.
- 애플리케이션 로딩 시점에 파싱하여 재사용 한다.

`단점`
- 엔티티가 지저분해진다.
- Spring data JPA 사용시 잘 사용하지 않는 기능이다.

#### Spring data JPA 에서 @NamedQuery 사용
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // JPQL에서 명확하게 namedParam을 사용했을때 @Param을 사용해야한다.
    // 애노테이션이 없어도 동작한다.
    // -> 먼저 엔티티명.메서드명의 NamedQuery가 존재하는지 먼저 찾고, 없다면 메서드명으로 쿼리를 생성한다.
    // 따라서 아래의 애노테이션은 생략이 가능하다.
//    @Query(name = "Member.findByUsername")
    List<Member> findByUsername(@Param("username") String username);

    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

- 기본적으로 @Query(name = "NamedQuery명") 으로 사용한다.
- 하지만 @Query 애노테이션은 생략이 가능하다.
    - 1차적으로 엔티티명.메소드 명으로 NamedQuery가 존재하는지 찾는다.
    - 만약 없다면 메소드명으로 쿼리를 생성한다.

> 실무에서는 잘 사용하지 않는 기능이다.
