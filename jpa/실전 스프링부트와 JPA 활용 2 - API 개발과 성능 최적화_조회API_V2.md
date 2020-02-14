# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_조회API_V2

#### 엔티티를 DTO로 변환
```java
/**
    * DTO로 변환하여 응답
    *
    * `문제점`
    * -> V1과 마찬가지로 Lazy Loading 으로 인한 쿼리가 너무 많이 나간다.
    *      -> N + 1 문제 발생
    *      -> fetchType EAGER 로 바꿔도 최적화 되지 않는다.
    *      -> 오히려 쿼리 예측이 더 힘들어진다.
    *  
    * @return
    */
@GetMapping("/api/v2/simple-orders")
public List<SimpleOrderDto> ordersV2 () {
    List<Order> orders = orderRepository.findAll(new OrderSearch());

    /* DTO로 변환 */
    List<SimpleOrderDto> orderDtos = orders.stream()
            .map(order -> new SimpleOrderDto(order))
            .collect(Collectors.toList());
    return orderDtos;
}

/**
    * API 스펙을 명확하게 정의해야 한다.
    */
@Data
static class SimpleOrderDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public SimpleOrderDto(Order order) {
        this.orderId = order.getId();
        this.name = order.getMember().getName(); // Lazy 초기화 (쿼리 발생)
        this.orderDate = order.getOrderDate();
        this.orderStatus = order.getStatus();
        this.address = order.getDelivery().getAddress(); // Lazy 초기화 (쿼리 발생)
    }
}
```

`문제점`
- N + 1 문제가 발생 한다.
- fetchType을 EAGER로 변경해도 최적화 되지 않는다.
    - 오히려 쿼리를 예측하기 더 힘들어 진다.

> 영속성 컨텍스트에서 먼저 조회하기 때문에 같은 식별자를 가리키고 있을경우 쿼리가 N + 1 만큼 발생하지 않지만, 최악의 경우로 계산하는것이 맞다.
