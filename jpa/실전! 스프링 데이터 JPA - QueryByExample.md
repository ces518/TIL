# 실전! 스프링 데이터 JPA - QueryByExample

#### QueryByExample
- Spring data JPA 최신 버전에 추가된 기능인데 Specification과 유사하다.
- 실무에서는 사용하기 어렵다.

```java
@Test
public void queryByExample() {
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

    // m1을 조회하고 싶은경우 ? -> 정적인 경우에 사용이 가능하다.
    memberRepository.findByUsername("m1");

    // Probe
    // 엔티티가 검색조건이 된다.
    Member member = new Member("m1");
    Team team = new Team("teamA");
    member.setTeam(team); // JOIN 같은 경우 컨디션 자체를 연관관계로 만든다.

    // 문제점 -> 도메인 객체를 가지고 검색조건을 만들어버린다.
    // 기본 전략 -> null은 무시함
    // age = 0 으로 기본값이 세팅되기때문에 검색 조건에 들어간다.
    // 이를 무시하는 코드가 필요함
    ExampleMatcher matcher = ExampleMatcher.matching()
            .withIgnorePaths("age");
    // ignore 설정한 matcher를 Example 생성시 같이 넣어준다.
    Example<Member> example = Example.of(member, matcher);

    List<Member> members = memberRepository.findAll(example); // Example을 파라메터로 받는걸 기본기능에서 제공


    // 이런 기술들의 문제점은 조인이 잘 해결이 되지 않는다.
    // QueryByExample은 조인이 되긴하지만 INNER JOIN만 가능하다.

    // then
    assertThat(members.size()).isEqualTo(1);
}
```
- Probe: 필드에 데이터가 있는 실제 도메인 객체
- ExampleMatcher: 특정 필드를 일치시키는 상세한 정보를 제공한다. 재사용 가능
- Exmaple: Probe와 ExampleMatcher로 구성, 쿼리를 생성하는데 생성한다.

- 검색 조건을 도메인 객체를 활용해서 생성한다.
- Example.of(Entity entity); 를 활용해서 생성한다.


`장점`
- 동적 쿼리르 편하게 처리할 수 있다.
- 도메인 객체를 그대로 사용한다.
- 데이터 저장소가 변경되어도 코드변경이 없도록 추상화 되어 있다.
- Spring data JPA `JpaRepository` 인터페이스에 포함 되어있다.

`단점`
- INNER JOIN만 가능하다.
- 중첩 제약조건이 안된다.
- 매칭 조건이 매우 단순하다.
    - equal 수준만 지원한다고 생각하면 된다.

`문제점`
- QueryByExample을 사용하면 도메인 객체를 활용하기 때문에 객체타입의 값이 null일 경우에는 검색조건에서 제외되지만
- 자바 기본타입은 기본값이 할당되기 때문에 검색조건에 포함되어 버린다.

`해결방법`
- ExampleMatcher 를 활용해서 검색조건에서 제외시킨다.

`한계점`
- 이런 기술들의 문제점은 조인이 잘 해결되지 않는다.
- QueryByExample은 조인이 되긴하지만 INNER JOIN 만 가능하다.

> 실무에서 사용하기에 매칭 조건이 너무 단순하다. QueryDSL을 사용할것
