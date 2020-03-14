# 실전! Querydsl - 기본 조인

#### 기본 조인
```java
/**
    * TeamA에 소속된 모든 회원을 찾아라.
    */
@Test
public void join() {
    List<Member> result = queryFactory
            .selectFrom(member)
            .join(member.team, team)
//                .leftJoin(member.team, team)
//                .rightJoin(member.team, team)
            .where(team.name.eq("teamA"))
            .fetch();

    // result 결과중 username 프로퍼티의 값을 검증
    // > 조회된 회원의 이름은 member1, member2 인지 검증
    assertThat(result)
            .extracting("username")
            .containsExactly("member1", "member2");
}
```
- 조인의 기본 문법은 첫 번째 파라미터에 조인 대상을 지정하고, 두번째 파라미터에 별칭으로 사용할 QType을 지정해준다.
- leftJoin, rightJoin 등 지원

#### 세타 조인
```java
/**
    * 회원이름이 팀이름과 같은 회원을 조회
    */
@Test
public void theta_join() {
    // 세타조인
    // 연관관계가 없어도 조인을 하도록 제공
    // 외부 조인이 불가능하다.
    // > 추후 추가된 조인 on 을 사용하면 외부 조인이 가능하다.
    em.persist(new Member("teamA"));
    em.persist(new Member("teamB"));

    List<Member> result = queryFactory
            .select(member)
            .from(member, team)
            .where(member.username.eq(team.name))
            .fetch();
}
```
- 연관관계가 없어도 조인을 할 수 있도록 제공한다.
- 카티시안곱 연산 (Cross Join)을 먼저 진행한뒤, where 절에서 필터링을 한다.
    - DB 내부에서 최적화를 하겠지만 이론상 성능이 좋지 않음
- 과거에는 외부조인이 불가능 했으나 최신 버전에서 추가된 조인 on 절을 이용해 외부조인이 가능하다.
