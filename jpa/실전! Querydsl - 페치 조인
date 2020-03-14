# 실전! Querydsl - 페치 조인

#### 페치 조인
- 페치 조인은 SQL에서 제공하는 기능은 아니다.
- SQL조인을 활용해서 연관된 엔티티를 SQL 한방에 조회하는 기능이다.
- 주로 성능 최적화시에 사용한다.

`페치조인 미적용시`
```java
@PersistenceUnit
EntityManagerFactory emf;

@Test
public void fetchJoinNo() {
    em.flush();
    em.clear();

    // 멤버와 팀의 관계는 지연로딩 LAZY이다.
    // 조회시 멤버만 조회되고, 팀은 조회되지 않음
    Member findMember = queryFactory
            .selectFrom(member)
            .where(member.username.eq("member1"))
            .fetchOne();

    // 로딩된 엔티티인지 를 판단
    boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
    assertThat(loaded).as("페치조인 미적용").isFalse();
}
```

- EntityManagerFactory를 통해 PersistenceUnitUtils을 가져올수 있음.
- 해당 Util을 이용해 특정 엔티티가 LAZY 로딩 초기화가 된 엔티티인지 판별이 가능하다.
- 페치조인을 적용하지 않았기 때문에 해당값은 false가 된다.

`페치조인 적용`
```java
@Test
public void fetchJoin() {
    em.flush();
    em.clear();

    // 멤버 조회시 연관된 엔티티인 팀을 같이 가져온다.
    Member findMember = queryFactory
            .selectFrom(member)
            .join(member.team, team).fetchJoin() // fetchJoin
            .where(member.username.eq("member1"))
            .fetchOne();

    // 로딩된 엔티티인지 를 판단
    boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
    assertThat(loaded).as("페치조인 적용").isTrue();
}
```

- 페치조인 적용시 join과 다른 문법들은 동일하고, 조인절 뒤에 fetchJoin()을 사용하면 페치조인이 적용된다.
    - 기본 JPA 페치조인과 동일하게 동작한다.
    - 컬렉션 페치조인시 주의점 등..
