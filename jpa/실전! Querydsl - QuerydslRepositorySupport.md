# 실전! Querydsl - QuerydslRepositorySupport

#### QuerydslRepositorySupport
- 메뉴얼에는 존재하지 않는다.
- Querydsl을 쓰기위해서 사용하는 추상클래스 이다.

```java
public class MemberRepositoryImpl extends QuerydslRepositorySupport implements MemberRepositoryCustom {
        ...
  public Page<MemberTeamDto> searchPageSimple2(MemberSearchCondition searchCondition, Pageable pageable) {

    // Querydsl 3버전에 만들어진거라 select절이 가장 마지막에 오게된다.
    // EntityManager도 주입을 받아줌
    JPQLQuery<MemberTeamDto> query = from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(searchCondition.getUsername()),
                    teamNameEq(searchCondition.getTeamName()),
                    ageGoe(searchCondition.getAgeGoe()),
                    ageLoe(searchCondition.getAgeLoe())
            )
            .select(new QMemberTeamDto(member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name));

    List<MemberTeamDto> content = getQuerydsl().applyPagination(pageable, query).fetch();
    long totalCount = query.fetchCount();

    return new PageImpl<>(content, pageable, totalCount);
  }
        ...
}
```

`장점`
- getQuerydsl().applyPagination()
        - 스프링 데이터가 제공하는 페이징을 Querydsl로 편리하게 변환 가능
- from() 으로 시작ㅇ이 가능하다.
        - 최근에는 select()로 시작하는 것이 더 명시적
- EntityManager 제공

`한계`
- Querydsl3.x 버전으로 만듬
- Querydsl4.x 에서 나온 JPAQueryFactory로 시작할 수 없음
        - select로 시작할 수 없음 (from으로 시작)
- QueryFactory를 제공하지 않음
- 스프링 데이터 Sort 기능이 정상 동작하지 않음

> 이 기능을 사용하면 Sort 기능은 직접 사용 해야한다.

> 메소드 체이닝으로 인한 코드 흐름이 끊긴다.. from() 절로 시작해야해서 가독성이 떨어진다.
