# 실전! 스프링 데이터 JPA - Specifications

#### Specifications
- 책 도메인 주도 설계는 Specification(명세) 라는 개념을 소개 한다.
- Spring data JPA는 JPA Criteria를 활용해서 이 개념을 사용할 수 있도록 지원 한다.

`Specification(명세)`
- where의 and or 등을 언어에 상관 없이 조립해서 사용하는 개념


`술어(predicate)`
- 참 또는 거짓으로 평가
- AND OR 같은 연산자로 조합해서 다양한 검색조건을 쉽게 생성 (컴포지트 패턴)
- -> 검색조건 하나 하나
- Spring data JPA는 `org.springframework.data.jpa.domain.Specification` 클래스로 정의

`명세 기능 사용 방법`
- `JpaSpecificationExecutor` 인터페이스를 상속 받는다.
```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom, JpaSpecificationExecutor<Member> {
    ...
}
```

- Spec을 정의한다.
```java
public class MemberSpec {

    public static Specification<Member> teamName(final String teamName) {

        return (Specification<Member>) (root, query, criteriaBuilder) -> {

            if (StringUtils.isEmpty(teamName)) {
                return null;
            }

            Join<Member, Team> team = root.join("team", JoinType.INNER);// 회원과 팀 조인
            return criteriaBuilder.equal(team.get("name"), teamName);
        };
    }

    public static Specification<Member> username(final String username) {
        return (Specification<Member>) (root, query, criteriaBuilder) -> criteriaBuilder.equal(root.get("username"), username);
    }
}
```

```java
@Test
public void specBasic() {
    // given
    Team teamA = new Team("teamA");
    em.persist(teamA);

    Member m1 = new Member("m1", 0, teamA);
    Member m2 = new Member("m2", 0, teamA);
    em.persist(m1);
    em.persist(m2);

    em.flush();
    em.clear();

    // when

    // Specification의 장점 기존에 정의해둔 스팩을 and, or등을 사용해서 조합이 가능하다.
    // Criteria를 활용한 기술
    // 실무에서 사용하기엔 너무 에로사항이 많다..
    // QueryDSL을 쓰도록 하자.
    Specification<Member> spec = MemberSpec.username("m1").and(MemberSpec.teamName("teamA"));
    List<Member> members = memberRepository.findAll(spec);

    // then
    assertThat(members.size()).isEqualTo(1);
}
```

- Specification은 기존에 정의 해둔 스펙을 and, or 등을 사용해 조합하여 사용이 가능하다.
- Criteria를 활용한 기술이다.
- 하지만 실무에선 사용하기 번잡하다..
- QueryDSL를 사용하는것을 권장.

> 알고 있으면 편리하게 쓸 수 있지만, 실무에서는 잘 쓰지 않는 기능이다.
