# 실전! 스프링 데이터 JPA - @Query 값, DTO 조회

#### @Query 값, DTO 조회
- 값, DTO를 조회하는 방법

`MemberRepository`
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // JPQL에서 명확하게 namedParam을 사용했을때 @Param을 사용해야한다.
    // 애노테이션이 없어도 동작한다.
    // -> 먼저 엔티티명.메서드명의 NamedQuery가 존재하는지 먼저 찾고, 없다면 메서드명으로 쿼리를 생성한다.
    // 따라서 아래의 애노테이션은 생략이 가능하다.
//    @Query(name = "Member.findByUsername")
    List<Member> findByUsername(@Param("username") String username);

    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);

    @Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);

    @Query("select m.username from Member m")
    List<String> findUsernames();

    // DTO 조회시 new operation을 사용해야 한다.
    @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
    List<MemberDto> findMemberDto();
}
```

`값 조회시`
- return 타입을 String, Long 등으로 지정하여 사용

`DTO 조회시`
- return 타입을 DTO로 지정하고, new Operation 을 사용해야한다.

> 두가지 모두 실무에서 자주 사용한다.
