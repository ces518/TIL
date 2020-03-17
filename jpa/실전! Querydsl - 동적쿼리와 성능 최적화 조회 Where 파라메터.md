# 실전! Querydsl - 동적쿼리와 성능 최적화 조회 Where 파라메터

#### 동적쿼리와 성능 최적화 조회 Where 파라메터
```java
/**
    * Where 파라메터를 활용한 동적쿼리 + 성능최적화
    * @param searchCondition
    * @return
    */
public List<MemberTeamDto> search(MemberSearchCondition searchCondition) {
    return queryFactory
            .select(new QMemberTeamDto(member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(searchCondition.getUsername()),
                    teamNameEq(searchCondition.getTeamName()),
                    ageGoe(searchCondition.getAgeGoe()),
                    ageLoe(searchCondition.getAgeLoe())
            )
            .fetch();
}

public List<Member> searchMember(MemberSearchCondition searchCondition) {
    // select 프로젝션이 달라져도 조건절을 재사용할 수 있다.
    return queryFactory
            .selectFrom(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(searchCondition.getUsername()),
                    teamNameEq(searchCondition.getTeamName()),
                    ageGoe(searchCondition.getAgeGoe()),
                    ageLoe(searchCondition.getAgeLoe())
            )
            .fetch();
}

/*
    Predicate 보다는 BooleanExpression을 사용하는것이 좋다.
    > 조립이 가능해진다.
*/
private BooleanExpression usernameEq(String username) {
    return StringUtils.isEmpty(username) ? null : member.username.eq(username);
}

private BooleanExpression teamNameEq(String teamName) {
    return StringUtils.isEmpty(teamName) ? null : team.name.eq(teamName);
}

private BooleanExpression ageGoe(Integer ageGoe) {
    return ageGoe == null ? null : member.age.goe(ageGoe);
}

private BooleanExpression ageLoe(Integer ageLoe) {
    return ageLoe == null ? null : member.age.loe(ageLoe);
}
```

- Where 파라메터를 활용한 동적 쿼리 + 성능최적화 처리
- 이 방법의 장점은 재사용이 가능하다는 점이다.
    - 메소드를 조합하여 조건절 조립이 가능해진다. (BooleanExpression)

> 상황에따라 Builder를 사용해야하는 경우도 있다. 기본으로는 Where 조건절 방법을 사용하고 때에 따라 Builder를 사용하는것을 권장
