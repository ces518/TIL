# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_컬렉션조회_V3

#### DTO로 변환 - 페치조인
- 엔티티를 DTO로 변환하는 작업을 페치조인으로 최적화
```java
@GetMapping("/api/v3/orders")
public List<OrderDto> ordersV3 () {
    List<Order> orders = orderRepository.findAllWithItem(new OrderSearch());
    List<OrderDto> orderDtos = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(Collectors.toList());

    return orderDtos;
}
/*
    Order와 OrderItem join시 중복된 데이터가 발생한다.
    JPA에서 Order 데이터가 뻥튀기 되어 버린다.
    > fetch join 은 DB 입장에서 select절에 데이터를 추가해주냐 마냐 차이일뿐 결국 join sql을 사용한다.
* */
public List<Order> findAllWithItem(OrderSearch orderSearch) {
    // distinct 키워드를 사용하면 DB distinct + JPA 에서 중복 엔티티를 제거한다.
    return em.createQuery(
            "select distinct o from Order o " +
                    "join fetch o.member m " +
                    "join fetch o.delivery d " +
                    "join fetch o.orderItems oi " +
                    "join fetch o.item i", Order.class)
            .getResultList();
}
```

- 페치조인으로 인해 SQL이 1번 실행된다.
- distinct 를 사용해 중복된 엔티티를 제거해 준다.
- 단점은 페이징이 불가능하다.
- 컬렉션 페치조인은 JPQL당 1개만 사용이 가능하다.
    - 데이터가 부정확할 수 있기 때문에 불가능..

- 페치 조인시 페이징은 페이징 쿼리가 나가지 않고, 애플리케이션 레벨에서 페이징 처리를 한다..
    - 모든 데이터를 조회한뒤 메모리에서 페이징을 진행하기 때문에 매우 위험하다.
