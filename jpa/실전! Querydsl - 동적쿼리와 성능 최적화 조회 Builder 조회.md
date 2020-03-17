# 실전! Querydsl - 동적쿼리와 성능 최적화 조회 Builder 조회

#### 동적쿼리와 성능 최적화 조회 Builder 조회

`MemberTeamDto`
```java
@Data
public class MemberTeamDto {

    private Long memberId;
    private String username;
    private int age;
    private Long teamId;
    private String teamName;

    @QueryProjection
    public MemberTeamDto(Long memberId, String username, int age, Long teamId, String teamName) {
        this.memberId = memberId;
        this.username = username;
        this.age = age;
        this.teamId = teamId;
        this.teamName = teamName;
    }
}
```

- 성능 최적화를 위해 사용할 DTO를 정의한다.
- 멤버와 팀 엔티티의 모든 정보를 가져오는것 보다는 프로젝션에 필요한 데이터만을 명시해서 가져오는것이 성능상 이점을 가진다.

`MemberSearchCondition`
```java
@Data
public class MemberSearchCondition {
    // 회원명, 팀명, 나이(Goe, Loe)

    private String username;
    private String teamName;
    private Integer ageGoe;
    private Integer ageLoe;
}
```
- 검색조건을 가지는 MemberSearchCondition 객체

`빌더를 사용한 성능최적화 + 동적쿼리`
```java
/**
    * 빌더를 이용해 성능최적화 + 동적쿼리
    * @param condition
    * @return
    */
public List<MemberTeamDto> searchByBuilder(MemberSearchCondition condition) {

    BooleanBuilder builder = new BooleanBuilder();
    if (StringUtils.hasText(condition.getUsername())) {
        builder.and(member.username.eq(condition.getUsername()));
    }
    if (StringUtils.hasText(condition.getTeamName())) {
        builder.and(team.name.eq(condition.getTeamName()));
    }
    if (condition.getAgeGoe() != null) {
        builder.and(member.age.goe(condition.getAgeGoe()));
    }
    if (condition.getAgeLoe() != null) {
        builder.and(member.age.loe(condition.getAgeLoe()));
    }

    return queryFactory
            .select(new QMemberTeamDto(member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name))
            .from(member)
            .leftJoin(member.team, team)
            .where(builder)
            .fetch();
}
```
- @QueryProjection을 사용하여 DTO를 이용한 조회
    - 이 방식의 단점은 순수한 DTO가 아닌 Querydsl에 의존하게 된다는 점이다.
- Builder를 이용해서 동적 쿼리를 처리하게되면 Mybatis(ibatis) 와 같은 모양새를 이루고 있게 된다.
    - 뭔가 객체지향 스럽지 않다.

#### 동적쿼리 작성시 주의점
```java
@Test
public void searchTestBuilder() throws Exception {
    // given
    Team teamA = new Team("teamA");
    Team teamB = new Team("teamB");
    em.persist(teamA);
    em.persist(teamB);

    Member member1 = new Member("member1", 10, teamA);
    Member member2 = new Member("member2", 20, teamA);

    Member member3 = new Member("member3", 30, teamB);
    Member member4 = new Member("member4", 40, teamB);

    em.persist(member1);
    em.persist(member2);
    em.persist(member3);
    em.persist(member4);

    // 가급적이면 이런 동적쿼리는 페이징 쿼리가 있는것이 좋다.
    // 조건절이 없다면 모든 데이터를 끌어오게 된다.
    // 데이터가 3만건이 존재한다면 3만건을 끌어오게 되어 문제가 발생함!
    MemberSearchCondition searchCondition = new MemberSearchCondition();
    searchCondition.setAgeGoe(35);
    searchCondition.setAgeLoe(40);
    searchCondition.setTeamName("teamB");

    // when
    List<MemberTeamDto> result = memberJpaRepository.searchByBuilder(searchCondition);

    // then
    assertThat(result)
            .extracting("username")
            .containsExactly("member4");
}
```
- 동적쿼리를 작성할 때 많이들 실수하는 부분은 동적쿼리 조건이 존재하지 않을때에 대한 처리이다.
- 개발 초기단계에서는 별 문제가 되지 않지만, 시간이 자나 데이터가 3만건, 30만건이 되었을때 문제가 된다.
- 만약 동적쿼리의 조건이 하나도 존재하지 않을경우 모든 데이터를 끌어오게된다.
    - 모든 데이터를 쿼리하게 된다.

> 가급적이면 이러한 동적쿼리는 페이징 쿼리가 존재하는것이 좋음. (최소한 limit은 존재할것)
