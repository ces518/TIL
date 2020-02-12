# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_조회API_V1

#### 지연 로딩과 조회 성능  최적화
- 주문 + 배송정보 + 회원 조회 API

#### 간단한 주문 조회 V1: 엔티티를 직접 노출
```java

/**
 * 컬렉션이 아닌 관계
 * xToOne
 * Order
 * Order -> Member
 * Order -> Delivery
 */
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {

    private final OrderRepository orderRepository;

    /**
     * `문제점`
     * 1.엔티티를 그대로 리턴하면, 무한 루프에 걸린다. (양방향 연관관계시)
     * -> 한쪽을 @JsonIgnore로 json 변환대상에서 제외시켜주어야함 (모든 양방향 연관관계)
     * 2.SimpleType.. 예외가 발생한다.
     * -> Lazy Loading 설정이 되어있을경우 실제 엔티티가 아닌 프록시 객체를 가지고 있다. (하이버네이트는 byteBuddy 사용)
     * -> json으로 변환을 하지 못함
     * -> Hibernate5Module 를 빈으로 등록 해야한다.
     * -> 즉시로딩으로 변경해서도 안된다.
     * 
     * * API 스펙에 그대로 노출하면 문제가 많다.
     * -> 반드시 필요한 데이터만 API를 통해 노출할것
     * * 성능상 문제도 있다.
     *
     * @return
     */
    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1 () {
        List<Order> all = orderRepository.findAll(new OrderSearch());

        // Hibernate5Module 을 사용하지않고 초기화 하는 방법
        for (Order order : all) {
            order.getMember().getName(); // Lazy Loading 강제 초기화
            order.getDelivery().getAddress(); // Lazy Loading 강제 초기화
        }
        return all;
    }
}
```

#### 문제점
- 1.엔티티를 그대로 노출하면 무한루프에 빠질 수 있다. (양방향 연관관계 설정시)
    - 한쪽은 @JsonIgnore로 json 변환대상에서 제외시켜주어야한다.
- 2.SimpleType.. 예외가 발생한다.
    - LazyLoading 설정이 되어있을경우 연관관계를 맺고있는 엔티티는 실제 엔티티가 아닌 프록시 객체를 가지고 있다. (Hibernate는 byteBuddy)
    - json으로 변환을 하지 못한다.
    - Hibernate5Module을 빈으로 등록하여 해결한다. (좋지 못한 방법,  DTO를 사용하는것이 더 좋다.)
    - 즉시로딩으로 변경해서는 안된다.

> 엔티티를 API 스펙에 그대로 노출하게 되면 문제가 많다. -> 반드시 필요한 데이터만 API를 통해 노출할것
