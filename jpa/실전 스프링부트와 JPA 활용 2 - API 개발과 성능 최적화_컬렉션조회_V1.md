# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_컬렉션조회_V1

#### 컬렉션 조회 최적화
- 주문내역에서 추가로 주문한 상품 정보를 추가로 조회한다.
- 1:N의 관계일경우 데이터가 뻥튀기 되어버려서 최적화시 고민을 많이 해야한다.

#### 주문조회 엔티티 직접 노출
```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1 () {
        List<Order> all = orderRepository.findAll(new OrderSearch());

        /* LAZY Loading */
        for (Order order : all) {
            order.getMember().getName();
            order.getDelivery().getAddress();

            List<OrderItem> orderItems = order.getOrderItems();
            orderItems.forEach(orderItem -> orderItem.getItem().getName());
        }
        return all;
    }
}
```

> 이전과 마찬가지로 엔티티를 그대로 노출하면 프록시 객체는 무시해 버리기 때문에 강제로 LazyLoading 초기화를 해준다.
