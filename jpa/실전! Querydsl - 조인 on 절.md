# 실전! Querydsl - 조인 on 절

#### 조인 on 절
- JPA 2.1 부터 지원한다.
- 1.조인 대상을 필터링
- 2.연관관계가 없는 엔티티를 외부 조인

##### 조인 대상 필터링

```java
/**
    * 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인, 회원은 모두 조회
    * JPQL: select m, t from Member m left join m.team t on t.name = 'teamA'
    * @throws Exception
    */
@Test
public void join_on_filtering() throws Exception {
    // select 절에 여러 타입이 존재하기 때문에 Tuple타입으로 반환됨.
    List<Tuple> result = queryFactory
            .select(member, team)
            .from(member)
            .leftJoin(member.team, team).on(team.name.eq("teamA"))
            .fetch();

    for (Tuple tuple : result) {
        System.out.println("tuple = " + tuple);
        /*
        teamA 소속은 정상적인 조인이 되고, teamB 소속은 조인에서 제외되고, leftJoin이기 때문에 멤버는 모두 조회
        tuple = [Member(id=3, username=member1, age=10), Team(id=1, name=teamA)]
        tuple = [Member(id=4, username=member2, age=20), Team(id=1, name=teamA)]
        tuple = [Member(id=5, username=member3, age=30), null]
        tuple = [Member(id=6, username=member4, age=40), null]
        * */
    }
}
```

> on 절을 활용해 조인 대상을 필터링 할 때, 외부조인이 아닌 내부 조인을 사용하면 where 절에서 필터링 하는것과 기능이 동일하다. 내부 조인일 경우 where로 사용하고, 정말 필요한 외부 조인일 경우에만 on 절을 사용한다.

##### 연관관계가 없는 엔티티 외부 조인
- 하이버네이트 5.1 부터 on 절을 사용해 서로 관계가 없는 필드로 외부 조인 이 가능하다 (내부조인도 가능)

```java
/**
    * 연관관계가 없는 엔티티를 외부 조인
    * 회원이름이 팀이름과 같은 회원을 조회
    */
@Test
public void join_on_no_relation() {
    em.persist(new Member("teamA"));
    em.persist(new Member("teamB"));

    List<Tuple> result = queryFactory
            .select(member, team)
            .from(member)
//                .leftJoin(member.team, team) 기존의 조인 문법 (연관관계로 조인을 하면 ID값으로 조인을 함)
            .leftJoin(team).on(member.username.eq(team.name))
            // 연관관계가 없는 조인 (일명 막 조인, ID값이 아닌 ON 절로만 조인)
            .fetch();

    for (Tuple tuple : result) {
        System.out.println("tuple = " + tuple);
        /*
        tuple = [Member(id=3, username=member1, age=10), null]
        tuple = [Member(id=4, username=member2, age=20), null]
        tuple = [Member(id=5, username=member3, age=30), null]
        tuple = [Member(id=6, username=member4, age=40), null]
        tuple = [Member(id=7, username=teamA, age=0), Team(id=1, name=teamA)]
        tuple = [Member(id=8, username=teamB, age=0), Team(id=2, name=teamB)]
        */
    }
}
```

`문법 주의`
- 일반 조인: `leftJoin(member.team ,team)`
- on 조인: `from(member).leftJoin(team).on(xxx)`

> 일반 조인과는 다르게 엔티티 하나만 들어간다.
