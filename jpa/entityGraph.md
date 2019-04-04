# Spring data jpa Entity Graph
두 엔티티가 1:N 단방향 관계일때 , Member의 List를 조회한다면
한개 이상의 쿼리가 발생한다.
이를 쿼리 튜닝하여 하나의 쿼리로 가져오고 싶다면
@EntityGraph를 활용하면된다.
```java
@Entity
@Getter @Setter
class Member {
    @Id @GeneratedValue
    Long id;
    
    String name;
    
    @ManyToOne
    Team team;
}

@Entity
@Getter @Setter
class Team {
    @Id @GeneratedValue
    Long id;
    
    String name;
}

interface MemberRepository extends JpaRepository<Member,Long> {
    @EntityGraph(attributePaths="team")
    List<Member> findAll();
}
```