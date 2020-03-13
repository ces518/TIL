# 실전! Querydsl - 페이징

#### 페이징
```java
@Test
public void paging() {
    // 데이터베이스 방언에 따라 다르게 실행된다.
    List<Member> members = queryFactory
            .selectFrom(member)
            .orderBy(member.username.desc())
            .offset(1)
            .limit(2)
            .fetch();

    assertThat(members.size()).isEqualTo(2);
}
```

- limit, offset을 지정하여 페이징이 가능하다.
- 만약 totalCount가 필요하다면 다음과 같이 fetchResults() 를 사용하자

```java
@Test
public void paging2() {
    // totalCount가 필요할 경우 fetchResults() 사용
    // 실무에서는 count쿼리와 페이징 쿼리가 다를경우가 있음
    // 그런경우에는 fetchResults() 보다는 count쿼리를 분리할것
    QueryResults<Member> results = queryFactory
            .selectFrom(member)
            .orderBy(member.username.desc())
            .offset(1)
            .limit(2)
            .fetchResults();
    long total = results.getTotal();
    List<Member> members = results.getResults();

    assertThat(total).isEqualTo(4);
    assertThat(results.getLimit()).isEqualTo(2);
    assertThat(results.getOffset()).isEqualTo(1);
    assertThat(members.size()).isEqualTo(2);
}
```

`fetchResults() 사용시 주의점`
- fetchResults()는 데이터 쿼리와, count쿼리가 동일하게 나간다.
    - 조인이 둘다 일치하게 나감.
- 실무에서는 성능 최적화를 위해 count 쿼리에서는 조인 조건을 제외시켜야함.
- 이런 경우에는 fetchResults()는 사용하지말것, count 쿼리를 별도의 쿼리로 분리하라.
- 실시간 데이터조회가 빈번한 API일수록 count쿼리는 별도로 분리하는것이 필수조건 이다.
