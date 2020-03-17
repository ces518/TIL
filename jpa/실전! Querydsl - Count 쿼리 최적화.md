# 실전! Querydsl - Count 쿼리 최적화

#### Count 쿼리 최적화
- 카운트 쿼리가 생략 가능한 경우 생략해서 처리
    - 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
    - 마지막 페이지 일 떄 (offset + 컨텐츠 사이즈를 더해서 전체 사이즈를 구함)

```java
/**
    * Count 쿼리 최적화
    * @param searchCondition
    * @param pageable
    * @return
    */
@Override
public Page<MemberTeamDto> searchPageComplexOptimization(MemberSearchCondition searchCondition, Pageable pageable) {
    List<MemberTeamDto> content = queryFactory
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
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();

    JPAQuery<Member> countQuery = queryFactory
            .select(member)
            .from(member)
            .where(
                    usernameEq(searchCondition.getUsername()),
                    teamNameEq(searchCondition.getTeamName()),
                    ageGoe(searchCondition.getAgeGoe()),
                    ageLoe(searchCondition.getAgeLoe())
            );

    return PageableExecutionUtils.getPage(content, pageable,  countQuery::fetchCount);
}
```
