# QueryDSL 소개
- http://www.querydsl.com

- QueryDSL은 JPQL을 자바코드로 작성이 가능하게 해준다.
- JPA 표준은 Criteria가 존재하지만 QueryDSL을 사용하는것이 좀 더 쉽고 편리하다.

> JPA를 사용하면서 생기는 이슈가 동적쿼리에 대한 처리이다.

- JPQL을 이용해 동적쿼리를 사용하려면 결국 문자열을 더하는 작업이 필요한데
- 이 과정에서 휴먼에러가 발생하기 쉽고, 타입 세이프하지 않다는 문제가 있다.


#### QueryDSL 적용
- QueryDSL 플러그인 및 라이브러리 설정이 필요하다.
- 아래는 QueryDSL 활용 동적쿼리 소스이다.

```java
public List<Order> findAll (OrderSearch orderSearch) {
    JPAQueryFactory query = new JPAQueryFactory(em);
    QOrder order = QOrder.order;
    QMEmber member = QMember.member;

    return query.select(order)
        .from(order)
        .join(order.member, member)
        .where(statusEq(orderSearch.getOrderStatus))
        .limit(1000)
        .fetch();
}

private BooleanExpression statusEq(OrderStatus statusCond) {
    if (statusCond == null) {
        return null;
    }
    return QOrder.order.status.eq(statusCond);
}
```

- 타입 세이프하고, 직관적인 코드작성이 가능하다.
- 동적인 쿼리를 BooleanExpression을 활용하여 깔끔한 조건절 처리가 가능하다.
    - null이 대입되면 조건절에 추가되지 않는다.
- 최종적으로 JPQL로 반환되어 실행된다.

`장점`
- 직관적인 문법
- 컴파일시점에 오류 체크
- 인텔리센스 지원
- 코드 재사용
- JPQL new 명령어와는 비교가 안될정도로 깔끔한 DTO 조회를 지원한다.

> Querydsl은 JPQL을 코드로 만드는 코드 빌더의 역할을 할 뿐이다. JPQL을 잘 이해하면 금방 배울 수 있으며 필수이다.
