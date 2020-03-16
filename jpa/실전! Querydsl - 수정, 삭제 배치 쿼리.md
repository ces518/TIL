# 실전! Querydsl - 수정, 삭제 배치 쿼리

#### 수정, 삭제 배치 쿼리
- 보통 JPA에서의 데이터 변경은 변경 감지 기능을 활용한다.
- 변경 감지 기능을 활용하면 개별 엔티티 건으로 나가기 때문에 쿼리가 많이 날아가게 된다.
    - 대량의 데이터를 한번에 수정하면 비효율적
- 이를 해결하기 위해 벌크연산 기능을 제공한다.

```java
@Test
public void bulkUpdate() {
    // 영향을 받은 로우수를 반환

    // member1 = 비회원
    // member2 = 비회원
    // member3 = 유지
    // member4 = 유지

    long count = queryFactory
            .update(member)
            .set(member.username, "비회원")
            .where(member.age.lt(28))
            .execute();

    // 벌크연산의 문제점
    // 영속성 컨텍스트가 아닌 DB에 바로 쿼리를 하는것이기 때문에
    // 영속성 컨텍스트에 존재하는 데이터와 일치하지 않을 수 있음.
    // 벌크연산 이후 flush를 해주어야 한다.

    // DB에서 조회를해도, 영속성 컨텍스트에 데이터가 있다면 영속성 컨텍스트가 우선권을 가진다.
    List<Member> result = queryFactory
            .select(member)
            .from(member)
            .fetch();
}
@Test
public void bulkAdd() {
    long count = queryFactory
            .update(member)
            .set(member.age, member.age.add(1))
            .execute();
}
```

- 쿼리 한번으로 대량 데이터 수정이 가능하다.

#### 벌크연산의 문제점
- 벌크연산의 경우에는 영속성 컨텍스트가 아닌 DB에 직접 쿼리를 하기때문에 영속성 컨텍스트에 존재하는 데이터와 일치하지 않을 수 있다.
- 벌크 연산 이후에는 flush(), clear()를 통해 영속성 컨텍스트를 초기화 해주어야 한다.

> DB에서 새롭게 엔티티를 조회 하더라도, 영속성 컨텍스트에 데이터가 존재한다면 영속성 컨텍스트에 존재하는 데이터가 우선권을 가지게 된다.
