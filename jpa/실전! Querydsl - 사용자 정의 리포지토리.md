# 실전! Querydsl - 사용자 정의 리포지토리

#### 사용자 정의 리포지토리
1. 사용자 정의 인터페이스 작성
2. 사용자 정의 인터페이스 구현
3. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속

```java
MemberRepository extends -> JpaRepository, MemberRepositoryCustom

MemberRepositoryImpl (MemberRepositoryCustom 의 구현체)
// MemberRepository에서 사용하기 때문에 MemberRepository + Impl 접미사의 명명규칙이 적용됨.
// 접미사 키워드는 변경이 가능하다.
```

`MemberRepository`
```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
    // 메소드 명으로 쿼리생
    List<Member> findByUsername(String username);
}
```

`MemberRepositoryCustom`
```java
public interface MemberRepositoryCustom {
    List<MemberTeamDto> search(MemberSearchCondition condition);
}
```

`MemberRepositoryImpl`
```java
// +Impl 이라는 접미사를 사용해주어야 한다.
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    @Override
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
}
```

- 모든 쿼리를 Custom에 때려박는 것은 좋지 않은 설계
- 핵심 비지니스로직, 엔티티에 집중된 것은 Repository, 화면에 특화된 경우 조회용 Repository를 생성 할 것

> 조회 쿼리가 너무 복잡하거나, 특정 API(화면)에 특화된 기능일 경우 `XXXQueryRepository` 처럼 별도의 리포지토리를 생성해서 관리하는것을 권장
