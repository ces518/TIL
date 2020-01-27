# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 주문 도메인 개발

#### 주문 도메인 개발
- 가장 중요한 도메인
- 비즈니스로직으로 얽혀서 돌아가는 부분을 JPA와 엔티티로 풀어내야한다..
- 트랜잭션스크립트패턴, 도메인모델패턴 을 많이 접해보지 못했봤을것
    - 엔티티내에 비즈니스로직이있고, 더 많은것을 엔티티로 풀어내는것이다

`구현 기능`
- 상품 주문
- 주문내역 조회
- 주문 취소

`순서`
- 주문 엔티티, 주문상품 엔티티 개발
- 주문 리포지토리 개발
- 주문 서비스 개발
- 주문 검색 기능
- 주문 기능 테스트

```java
@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {

    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    /*
    @Temporal(TemporalType.TIMESTAMP)
    private Date date;
     */
    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status; // 주문상태: ORDER, CANCEL

    /* 연관관계 편의 메소드
    *  연관관계 편의 메소드의 위치는 핵심 적인 엔티티에 두는것이 좋다.
    * */
    public void setMember (Member member) {
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem (OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery (Delivery delivery) {
        this.delivery = delivery;
        delivery.setOrder(this);
    }

    //== 생성 메소드 ==//
    // 생성 메소드를 사용하는것이 중요함
    // 여기저기 비즈니스로직을 찾아다니지 않아도되고, 변경이 일어나더라도 한곳에서 관리가 된다.

    /**
     * 주문생성에 대한 비즈니스로직을 응집해 두는것
     */
    public static Order createOrder (Member member, Delivery delivery, OrderItem... orderItems) {
        Order order = new Order();
        // 주문자 세팅
        order.setMember(member);
        // 딜리버리 세팅
        order.setDelivery(delivery);

        // 주문상품 세팅
        for (OrderItem orderItem : orderItems) {
            order.addOrderItem(orderItem);
        }
        // 주문상태: 주문으로 세팅
        order.setStatus(OrderStatus.ORDER);
        // 주문시간을 현재로 세팅
        order.setOrderDate(LocalDateTime.now());
        return order;
    }

    //== 비즈니스 로직 ==//
    /**
     * 주문 취소
     * - 배송완료된 상품은 취소하지 못하다는 비즈니스로직
     */
    public void cancel () {
        if (delivery.getStatus() == DeliveryStatus.COMP) {
            throw new IllegalStateException("이미 배송완료된 상품은 취소할 수 없습니다.");
        }
        // 상태를 취소로 변경
        this.setStatus(OrderStatus.CANCEL);
        // 상품의 재고를 원복한다.
        for (OrderItem orderItem : orderItems) {
            orderItem.cancel();
        }
    }

    //== 조회 로직 ==//
    // 뭔가 계산이 필요할때..

    /**
     * 전체 주문 가격 조회
     */
    public int getTotalPrice () {
        return orderItems.stream()
                .mapToInt(OrderItem::getTotalPrice)
                .sum();
        /*
        int totalPrice = 0;

        for (OrderItem orderItem : orderItems) {
            totalPrice += orderItem.getTotalPrice();
        }
        return totalPrice;
        */
    }
}
@Entity
@Getter @Setter
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name ="item_id")
    private Item item;

    private int orderPrice; // 주문가격 (주문당시 가격)

    private int count; // 수량

    //== 생성 메서드 ==//
    public static OrderItem createOrderItem (Item item, int orderPrice, int count) {
        // orderPrice를 Item의 price를 참조하지않고 따로 받아오는 이유는 할인을 하는등의 상황등에 대처하기 위함
        OrderItem orderItem = new OrderItem();
        orderItem.setItem(item);
        orderItem.setOrderPrice(orderPrice);
        orderItem.setCount(count);

        // 구매하는것 만큼 재고를 감소시킨다.
        item.removeStock(count);

        return orderItem;
    }

    //== 비즈니스 로직 ==//
    /**
     * 재고수량을 원복한다.
     */
    public void cancel() {
        // 재고수량을 원복한다.
        getItem().addStock(this.count);
    }

    //== 조회 로직 ==//

    /**
     * 주문상품 전체 가격 조회
     */
    public int getTotalPrice() {
        return getOrderPrice() * getCount();
    }
}
```

- 생성메소드를 만들어, 엔티티생성시 복잡한 비즈니스로직을 엔티티 내부에 위치하였다.
    - 비즈니스 로직을 찾기위해 여기저기 뒤져야하는 수고를 덜고, 비즈니스로직이 수정되었을때 관리를 한곳에서만 하기 위함이다.

> 도메인과 관련된 비즈니스로직을 엔티티 내부에 위치시킴으로써 객체지향적이고 응집도를 높힌다.
