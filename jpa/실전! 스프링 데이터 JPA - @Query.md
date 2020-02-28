# 실전! 스프링 데이터 JPA - @Query

#### @Query, 리포지토리에 바로 정의하기
- @Query 애노테이션을 사용하면 리포지토리에서 바로 JPQL을 정의하여 사용할 수 있다.

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
}
```

`장점`
- 애플리케이션 로딩 시점에 파싱하여 오류 체크가 가능하다.
    - 이름이 없는 NamedQuery라고 생각하면 된다.
