# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_조회API_V3

#### 페치조인 최적화
- DTO로 변환을 했지만 N + 1 문제로 인한 성능 최적화
```java
@GetMapping("/api/v3/simple-orders")
public List<SimpleOrderDto> ordersV3 () {
    /* fetchJoin 사용 */
    List<Order> orders = orderRepository.findAllWithMemberDelivery();

    /* DTO로 변환 */
    List<SimpleOrderDto> orderDtos = orders.stream()
            .map(order -> new SimpleOrderDto(order))
            .collect(Collectors.toList());
    return orderDtos;
}

/**
    * 한방 쿼리로 멤버와, 딜리버리를 함께 조회한다.
    * @return
    */
public List<Order> findAllWithMemberDelivery() {
    return em.createQuery("select o from Order o " +
            "join fetch o.member m " +
            "join fetch o.delivery d", Order.class)
            .getResultList();
}
```

- 페치조인은 연관관계 설정을 모두 무시하고 런타임시에 동적으로 데이터를 조회 한다.
    - 연관관계가 LAZY 라도 페치조인을 한다면 연관관계를 맺고 있는 엔티티를 함께 조회한다.

`주의점`
- 1:N 컬렉션 조인시 페이징을 하지말것 (WARN 경고를 띄우고 애플리케이션 레벨에서 페이징을 해주지만 메모리 문제떄문에 매우 위험)
- 1:N 컬렉션 조인시 중복된 데이터가 나타날 수 있다.
    - 카티시안 곱이 발생
    - DISTINCT로 해결이 가능하다.
