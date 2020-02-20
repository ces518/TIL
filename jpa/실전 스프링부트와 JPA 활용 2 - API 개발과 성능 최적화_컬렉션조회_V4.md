# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_컬렉션조회_V4

#### V4 - DTO 로 바로 조회
```java
@GetMapping("/api/v4/orders")
public List<OrderQueryDto> ordersV4 () {
    return orderQueryRepository.findOrderQueryDtos();
}
```

`OrderQueryRepository를 사용한 이유 ?`
- OrderQueryRepository를 별도로 분리하여 사용한다.
    - DTO로 바로 조회하는것은 특정 화면에 종속되어 있기때문에 다른 곳에서 재사용하기 힘들며 순수한 Repository 계층에 맞지 않다.
    - 영속성 컨텍스트에서 관리되지 않아, DirtyChecking 등이 되지 않는다. (비즈니스 로직을 사용하기 어렵고, 사실상 단순 조회만 존재한다.)
    - 따라서 따로 분리하여 관리하는것이 좋다.

```java
@Data
public class OrderQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
//        this.orderItems = orderItems;
    }
}
@Data
public class OrderItemQueryDto {

    private Long orderId;
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```

`OrderDto와 OrderItemDto를 따로 분리한 이유 ?`
- 분리한 이유는 크게 2가지가 있다.
- 1.컨트롤러에 OrderDto가 존재하지만, Repository에서 이를 참조할 경우 순환참조가 형성된다. (이는 잘못된 구조이다.)
- 2.Repository에서 사용될 객체이며 Repository에서 이를 알아야 하기 때문에 OrderQueryRepository와 같은 패키지에 새롭게 정의해주었다.

#### 정리
- 쿼리
    - 루트 1번, 컬렉션 N번 실행
- ToOne 관계를 먼저 조회하고, ToMany 관계를 각각 별도로 조회한다.
    - ToMany관계는 데이터가 뻥튀기 되기 때문에 사용한 방식
- row 수가 증가하지 않는 ToOne관계는 조인으로 최적화하고, ToMany관계는 최적화하기 힘드므로 별도의 쿼리로 채워 넣어준다.

> 이 방법은 N + 1 문제가 발생한다.
