# 실전! 스프링 데이터 JPA - @EntityGraph

#### fetch join
- EntityGraph 를 이해하려면 jpa fetch join을 이해해야 한다.
- 먼저 지연로딩에 대한 이해가 필요하다.

#### 지연로딩에 대한 이해
```java
// given

// member1 -> teamA
// member2 -> teamB
// member와 team의 연관관계는 LAZY이다.
// 실무에서는 무조건 LAZY로 지정해야한다.
// 가짜 객체를 담아뒀다가 Team을 사용할때 Team을 조회한다.
Team teamA = new Team("teamA");
Team teamB = new Team("teamB");
teamRepository.save(teamA);
teamRepository.save(teamB);


Member member1 = new Member("member1", 10, teamA);
Member member2 = new Member("member2", 10, teamB);
memberRepository.save(member1);
memberRepository.save(member2);

em.flush();
em.clear();

// when
List<Member> members = memberRepository.findAll();

// LAZY로 지정했기 때문에
// N + 1 문제가 발생한다.
// JPA에서는 이를 해결하기 위해 fetch join 기능을 제공한다.
for (Member member : members) {
    System.out.println("member = " + member.getUsername());
    // LAZY로 지정했기 때문에 team은 프록시 객체이다
    // member.team = class study.datajpa.entity.Team$HibernateProxy$JDELlbBI
    System.out.println("member.team = " + member.getTeam().getClass());
    System.out.println("member.team = " + member.getTeam().getName());
}
@Entity
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
}
```

- Member 엔티티와 Team 엔티티의 연관관계는 LAZY로 지정한다.
    - LAZY로 지정하면 Member 엔티티를 조회할때 가짜 객체를 담아뒀다가 Team을 사용할때 Team엔티티를 조회하는 쿼리가 발생한다.
    - 이를 **N + 1 문제** 라고한다.
    - JPA에서는 이를 해결하기 위해 fetch join 기능을 제공한다.

> 연관관계 설정시 실무에서는 반드시 LAZY로 지정해야함 !

#### fetch join
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m left join fetch m.team")
    List<Member> findMemberFetchJoin();
}
// fetch join을 사용하면 Member 엔티티를 조회할때 Team 엔티티까지 함께 조회해온다.
List<Member> fetchMembers = memberRepository.findMemberFetchJoin();

for (Member member : fetchMembers) {
    System.out.println("member = " + member.getUsername());
    System.out.println("member.team = " + member.getTeam().getClass()); // 프록시 객체가 아닌 실제 Team 객체를 가져온다.
    System.out.println("member.team = " + member.getTeam().getName());
}
```

- fetch join을 사용하면 Member 엔티티 조회시 Team 엔티티까지 함께 조회해 온다.
- 따라서 N + 1 문제가 해결됨

> fetch join을 사용하려면 매번 JPQL을 작성해야한다. 이런 번거로움을 해결하기위해 JPA 에서 EntityGraph 라는 기능 제공

#### EntityGraph
- JPA 표준 스펙 2.2 부터 제공하는 기능
- fetch join을 편리하게 할때 사용한다.

```java
/*
    @EntityGraph
    - fetch join을 간단하게 사용하고 싶을때 사용
    - attributePaths 에 해당 속성을 지정해주면 된다.
    아래의 세가지 방식을 지원한다.

    > EntityGraph는 JPA 2.2 부터 제공하는 기능이다.
    -> NamedEntityGraph 라는 기능도 존재함
    -> NamedQuery와 유사하다.

    * 간단한 fetch join의 경우 EntityGraph를 사용하고 복잡해 지는경우 JPQL 혹은 QueryDSL 사용
*/
@Override
@EntityGraph(attributePaths = "team")
List<Member> findAll();

@EntityGraph(attributePaths = "team")
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

//    @EntityGraph(attributePaths = "team")
@EntityGraph("Member.All") // NamedEntityGraph 기능 사용
List<Member> findEntityGraphByUsername(@Param("name") String username);
```

- Spring data JPA에서는 @EntityGraph 애노테이션으로 쉽게 사용할 수 있다.
    - attributePaths 속성에 연관관계 필드를 명시해 주면된다.

`@EntityGraph의 세가지 사용 방법`
- JpaRepositroy 의 기본 메소드를 Override 하여 해당 메소드에서 사용
- @Query를 사용하여 JPQL과 함께 사용
- NamedEntityGraph 기능을 사용하여 미리 지정해둔 EntityGraph를 사용

#### NamedEntityGraph
```java
@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = { "id", "username", "age" }) // 연관관계 필드는 무한루프에 빠질수 있기때문에 ToString대상에서 제외할것
@NamedQuery(
        name = "Member.findByUsername",
        query = "select m from Member m where m.username = :username"
)
@NamedEntityGraph(name = "Member.All", attributeNodes = @NamedAttributeNode("team"))
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
    ...
}
```

- NamedQuery와 유사하며, 엔티티 상단에 정의해두고 name을 호출하여 사용하는 방식

#### 팁
- 간단한 fetch join의 경우 EntityGraph를 사용하고 복잡해 지는경우 JPQL 혹은 QueryDSL 사용하는것을 추천
