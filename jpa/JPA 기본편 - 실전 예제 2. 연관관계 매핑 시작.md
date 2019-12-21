# JPA 기본편 - 실전 예제 2. 연관관계 매핑 시작

#### 테이블 구조
- 이전과 같다.

#### 객체 구조
- 참조를 사용하도록 변경

- Member
    - id: Long
    - name: String
    - city: String
    - street: String
    - zipcode: String
    - orders: List
- Order
    - id: Long
    - member: Member
    - orderItems: List
    - orderDate: Date
    - status: OrderStatus
- OrderItem
    - id: Long
    - item: Item
    - order: Order
    - orderPrice: int
    - count: int
- Item
    - id: Long
    - name: String
    - price: int
    - stockQuantity: int

> 거의 대부분의 연관관계를 참조를 통해 접근이 가능하도록 변경

* 단방향 연관관계를 매핑하는것이 중요하다.
- 연관관계 주인을 누구로 정할것인가 ? 
    - 외래키가 있는쪽을 외래키 주인으로 생각하라

설계단계에서 단방향 매핑을 먼저하고, 후에 양방향이 필요할때 매핑을 해준다.

대부분의 경우에는 member에 orders를 넣는것이 좋은 케이스는 아니다.
- 관심사를 끊어야할것을 못끊어낸것 -> 잘못된 설계
- 비지니스 마다 다른 경우가 있긴하지만 특이한 케이스

> 양방향 연관관계는 존재하지 않아도 매핑하는데 전혀 문제가없다. (매핑시에는 단방향 연관관계만이 영향을 미친다.)

#### 양방향 연관관계를 만드는 이유
- 개발상의 편의 혹은 비지니스상 편하게 위해 만드는 것이지 중요한것은 아니다.
- 할수 있다면 최대한 단방향으로 하는것을 추천한다.

#### 설계후 엔티티
```java
@Entity
public class Item {
    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    private String name;
    private int price;
    private int stockQuantity;
}

@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String name;
    private String city;
    private String street;
    private String zipcode;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}

@Entity
@Table(name = "ORDERS")
public class Order {
    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;
//    @Column(name = "MEMBER_ID")
//    private Long memberId;
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();
    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    // 연관관계 편의 메소드
    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }
}

@Entity
public class OrderItem {
    @Id @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;
//    @Column(name = "ORDER_ID")
//    private Long orderId;
    @ManyToOne
    @JoinColumn(name = "ORDER_ID")
    private Order order;
//    @Column(name = "ITEM_ID")
//    private Long itemId;
    @ManyToOne
    @JoinColumn(name = "ITEM_ID")
    private Item item;
    private int orderPrice;
    private int count;
}

public enum OrderStatus {
    ORDER, CANCEL;
}
```
