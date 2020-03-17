# 실전! Querydsl - 스프링 데이터 페이징 활용
- 스프링 데이터의 Page, Pageable을 활용
- 전체 카운트를 한번에 조회하는 단순한 방법
- 데이터 내용과 전체 카운트를 별도로 조회하는 방법

#### 사용자 정의 인터페이스에 2가지 메소드 추가
```java
public interface MemberRepositoryCustom {

    List<MemberTeamDto> search(MemberSearchCondition condition);

    Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable);

    // 카운트 쿼리를 별도의 쿼리로 사용
    Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable);
}
```

`searchPageSimple()`
```java
@Override
public Page<MemberTeamDto> searchPageSimple(MemberSearchCondition searchCondition, Pageable pageable) {
    QueryResults<MemberTeamDto> results = queryFactory
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
            .fetchResults();

    // fetchResults() 를 사용하면, 컨텐츠 쿼리와 카운트 쿼리가 나간다.
    // orderBy는 카운트쿼리에서 제거됨
    // -> 단점: 카운트 쿼리 최적화를 못한다.
    List<MemberTeamDto> content = results.getResults();
    long totalCount = results.getTotal();

    return new PageImpl<>(content, pageable, totalCount);
}
```

- simple버전은 Querydsl의 fetchResults()를 활용해서 페이징 처리를 한다.
- orderBy 구문은 카운트 쿼리에서 자동으로 사라짐

> 단점: 카운트 쿼리 최적화를 하지 못한다.

`searchPageComplex()`
```java
/**
    * 쿼리 자체를 분리하는 방법
    * @param searchCondition
    * @param pageable
    * @return
    */
@Override
public Page<MemberTeamDto> searchPageComplex(MemberSearchCondition searchCondition, Pageable pageable) {
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

    // count 쿼리에서 성능최적화를 할 수있다.
    // 조인수를 줄이는 등..
    long totalCount = queryFactory
            .select(member)
            .from(member)
//                .leftJoin(member.team, team)
            .where(
                    usernameEq(searchCondition.getUsername()),
                    teamNameEq(searchCondition.getTeamName()),
                    ageGoe(searchCondition.getAgeGoe()),
                    ageLoe(searchCondition.getAgeLoe())
            )
            .fetchCount();

    return new PageImpl<>(content, pageable, totalCount);
}
```

- 별도의 쿼리로 분리하는 방법은 Count쿼리에서 성능 최적화를 진행할 수 있다.

> 데이터수가 많지 않은 경우에는 fetchResult()를 사용할것.. 의미없는 최적화이다.
