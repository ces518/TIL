# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_컬렉션조회_V2

#### 주문 조회 V2 DTO로 변환
```java
@GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2 () {
    List<Order> orders = orderRepository.findAll(new OrderSearch());
    List<OrderDto> orderDtos = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(Collectors.toList());

    return orderDtos;
}

@Data
static class OrderDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItem> orderItems;

    public OrderDto(Order o) {
        this.orderId = o.getId();
        this.name = o.getMember().getName();
        this.orderDate = o.getOrderDate();
        this.orderStatus = o.getStatus();
        this.address = o.getDelivery().getAddress();
        o.getOrderItems().forEach(o -> o.getItem().getName());
        this.orderItems = o.getOrderItems();
    }
}
```
- DTO 안에 엔티티가 있으면 안된다.
    - > 결국 엔티티가 노출 되는 꼴이기 때문에 별반 차이가 없음..

> OrderItem도 DTO로 모두 바꾸어 줘야한다.

#### 변경 후
```java
@GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2 () {
    List<Order> orders = orderRepository.findAll(new OrderSearch());
    List<OrderDto> orderDtos = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(Collectors.toList());

    return orderDtos;
}

@Data
static class OrderDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemDto> orderItems;

    public OrderDto(Order o) {
        this.orderId = o.getId();
        this.name = o.getMember().getName();
        this.orderDate = o.getOrderDate();
        this.orderStatus = o.getStatus();
        this.address = o.getDelivery().getAddress();
        this.orderItems = o.getOrderItems().stream()
                                            .map(orderItem -> new OrderItemDto(orderItem))
                                            .collect(Collectors.toList());
    }
}

@Data
static class OrderItemDto {

    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemDto(OrderItem orderItem) {
        this.itemName = orderItem.getItem().getName();
        this.orderPrice = orderItem.getOrderPrice();
        this.count = orderItem.getCount();
    }
}
```
