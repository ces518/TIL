# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_컬렉션조회_V6

#### V6 - DTO로 바로 조회 한방 쿼리로 최적화

```java
@GetMapping("/api/v6/orders")
public List<OrderFlatDto> ordersV6 () {
    return orderQueryRepository.findAllByDto_flat();
}
```

- join을해서 중복된 Order를 모두 출력함
- join된 결과 row를 그대로 내보낸다.

`장점`
- 쿼리가 한방으로 나간다.
- 페이징이 가능하다.

`단점`
- 중복 데이터가 존재한다.


#### OrderQueryDto로 변환
```java
@GetMapping("/api/v6/orders")
public List<OrderQueryDto> ordersV6 () {
    List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();

    return flats.stream()
            // Order에 해당하는 데이터를 OrderQueryDto로 객체로 변환하여 Key로 사용,
            // OrderItems에 해당하는 데이터를 OrderItemQueryDto로 변환하여 List형태로 value값으로 매핑한다.
            // OrderId가 같다면 같은 키로 간주
            .collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(),o.getName(), o.getOrderDate(), o.getOrderStatus(), o.getAddress()),
                    mapping(o -> new OrderItemQueryDto(o.getOrderId(),o.getItemName(), o.getOrderPrice(), o.getCount()), toList())
            )).entrySet().stream()
            // e.getKey() 는 OrderQueryDto이다.
            // 최종적으로 OrderQueryDto [OrderItemsDto] 형태로 반환됨
            .map(e -> new OrderQueryDto(e.getKey().getOrderId(),
                    e.getKey().getName(), e.getKey().getOrderDate(), e.getKey().getOrderStatus(),
                    e.getKey().getAddress(), e.getValue()))
            .collect(toList());
}
```

- OrderFlatDto를 루프를 돌면서 OrderQueryDto스펙에 맞게 데이터를 가공해준다.
- 단점은 애플리케이션에서 처리하는 로직이 많아짐
- 중복 데이터가 존재하기 때문에 V5보다 느려질 수 있다.
