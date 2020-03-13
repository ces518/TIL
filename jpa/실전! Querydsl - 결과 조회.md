# 실전! Querydsl - 결과 조회

#### 결과 조회
- fetch()
    - 리스트로 조회, 데이터가 없다면 빈 리스트 반환
- fetchOne()
    - 단건 조회
    - 결과가 없다면 null
    - 결과가 둘 이상이면 com.querydsl.core.NonUniqueResultException
- fetchFirst()
    - limit(1).fetchOne() 으로 동작
- fetchResults()
    - 페이징 쿼리를 같이 날린다.
    - totalCount 쿼리 추가 실행
- fetchCount()
    - count 쿼리로 변경하여 count수를 조회

```java
@Test
public void resultFetch() {
    // Member의 목록을 리스트로 조회
    List<Member> fetch = queryFactory
            .selectFrom(member)
            .fetch();

    // 단건 조회
    Member fetchOne = queryFactory
            .selectFrom(QMember.member)
            .fetchOne();

    // fetchFirst
    Member fetchFirst = queryFactory
            .selectFrom(QMember.member)
            //.limit(1).fetchOne() 과 동일
            .fetchFirst();

    // fetchResults()
    // 쿼리가 2번 실행된다.
    QueryResults<Member> results = queryFactory
            .selectFrom(QMember.member)
            .fetchResults();
    long totalCount = results.getTotal();// totalCount
    List<Member> content = results.getResults(); // 실제 데이터
    long limit = results.getLimit(); // limit
    long offset = results.getOffset(); // offset

    // 실무에서는 실제 데이터를 가져오는 쿼리와, totalCount쿼리가 다른 케이스가 있음
    // 따라서 성능 문제가 발생할 수 있음
    // 정말 실시간 데이터가 중요한 페이징 API의 경우에는 fetchResults()를 사용해선 안된다.


    // fetchCount()
    // select절을 count로 변경해서 날림
    // JPQL에서 엔티티를 직접 지정하면 식별자로 변경되어 실행된다.
    // 실제 SQL에서는 member_id 로 지정됨
    long fetchCount = queryFactory
            .selectFrom(member)
            .fetchCount();

}
```

> 실무에서는 실제 데이터를 가져오는 쿼리와, totalCount쿼리가 다른 케이스가 있음. 성능 문제가 발생할 수 있으며 실시간 데이터가 중요한 페이징 API의 경우에는 fetchResults()를 사용해선 안된다.
