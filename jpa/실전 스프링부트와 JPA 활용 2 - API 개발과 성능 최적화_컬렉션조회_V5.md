# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_컬렉션조회_V5

#### V5 - DTO로 바로조회 컬렉션 조회 최적화
```java
@GetMapping("/api/v5/orders")
public List<OrderQueryDto> ordersV5 () {
    return orderQueryRepository.findAllByDto_optimization();
}
```

```java
/**
    * 최적화 버전
    * - > 쿼리 두번으로 최적화 된다.
    * @return
    */
public List<OrderQueryDto> findAllByDto_optimization() {
    List<OrderQueryDto> result = findOrders();

    // orderId 목록으로 추출
    List<Long> orderIds = toOrderIds(result);

    Map<Long, List<OrderItemQueryDto>> orderItemMap = findOrderItemMap(orderIds);

    // orderItems 세팅
    result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));
    return result;
}

private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
    // inQuery로 OrderItem 목록 조회
    List<OrderItemQueryDto> orderItems = em.createQuery(
            "select new jpabook.jpashop.repositories.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count) " +
                    "from OrderItem oi " +
                    "join oi.item i " +
                    "where oi.order.id in :orderIds", OrderItemQueryDto.class)
            .setParameter("orderIds", orderIds)
            .getResultList();


    // OrderId 를 기반으로 Map으로 변환
    return orderItems.stream()
            .collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId));
}

private List<Long> toOrderIds(List<OrderQueryDto> result) {
    return result.stream()
                    .map(o -> o.getOrderId())
                    .collect(Collectors.toList());
}
```

- Query
    - 루트 1번, 컬렉션 1번
- ToOne 관계를 먼저 조회하고, orderIds를 추출하여 ToMany관계를 in Query로 한번에 조회한다.
- MAP을 사용해 매칭 성능을 향상 O(1)

#### 트레이드 오프
- 컬렉션 페치 조인과 비교 했을때, 많은 양의 코드를 직접 작성해야한다.
- 하지만 장점은 필요한 데이터만 조회하기 때문에, 네트웤 트래픽이 줄어든다.
