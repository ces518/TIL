# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 주문 서비스 개발

#### 주문 서비스 개발
- 주문 기능
- 주문 취소 기능
- 검색 기능

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderService {
    private final OrderRepository orderRepository;
    private final MemberRepository memberRepository;
    private final ItemRepository itemRepository;
    /**
     * 주문 기능
     */
    @Transactional
    public Long order (Long memberId, Long itemId, int count) {
        // 회원아이디, 상품아이디, 수량 만 넘어오기때문에 회원 리포지토리와 상품 리포지토리에서 엔티티 조회가 필요하다.
        Member findMember = memberRepository.find(memberId);
        Item findItem = itemRepository.findOne(itemId);

        // 배송정보
        // 단순 기능이기때문에 회원의 주소로 바로 세팅해준다.
        Delivery delivery = new Delivery();
        delivery.setAddress(findMember.getAddress());


        /**
         * 현재는 생성자 메소드를 이용해서 개발했지만
         * 다른 누군가는 new 로 생성하여 setter로 직접 값을 세팅하는방식으로 개발을 할 수 있다.
         * 그렇게 되면 생성로직을 변경하거나 수정등 변동이 일어날때 유지보수가 어렵게 된다.
         * 따라서 생성자를 protected로 제한하여 생성자 메소드만 사용하게끔 제한해 주어야한다.
         * -> JPA는 protected 생성자까지 허용한다.
         * 코드를 제약하는 스타일로 짜는것이 좋은스타일과 설계로 이끌어갈 수 있다.
         */
        // 주문 상품
        OrderItem orderItem = OrderItem.createOrderItem(findItem, findItem.getPrice(), count);

        // 주문
        Order order = Order.createOrder(findMember, delivery, orderItem);

        // 주문 저장
        // cascade 옵션 때문에 delivery, orderItem 이 모두 persist 된다.
        // 다른 곳에서 참조하지 않는경우에만 사용할것.
        // 라이프사이클을 공유할때 이점이 존재함.
        orderRepository.save(order);

        return order.getId();
    }

    /**
     * 주문 취소
     */
    @Transactional
    public void cancelOrder (Long orderId) {
        // 엔티티 조회
        Order findOrder = orderRepository.findById(orderId);
        // 주문취소
        findOrder.cancel();

        // Mybatis 와 같은 SQL 을 직접다루는 라이브러리들을 사용하면
        // 주문 취소가 일어났을때 이로 인해 발생한 데이터 변경점들을 직접 SQL로 날려줘야한다.
        // 이런 스타일의 개발은 Service 계층에 비즈니스로직들이 난잡하게 나올수 밖에 없다..
        // 하지만 JPA를 사용하면 엔티티의 데이터만 변경해주어도 트랜잭션 커밋시점에 이를 감지하여
        // 각각의 repository에 모두 변경사항을 반영해준다.
        // -> JPA 사용할때의 강점이다.
    }

    /**
     * 검색
     */
    /*
    public List<Order> findOrders (OrderSearch orderSearch) {
        return null;
    }
    */
}
```

`도메인 모델패턴`
- 엔티티에 비지니스 로직이 존재 한다.
- 서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할만 한다.
- 객체지향의 특성을 적극활용하는것을 도메인모델 패턴이라고 한다.
``

`트랜잭션 스크립트 패턴`
- 엔티티에 비즈니스 로직이 거의 없다. (getter, setter만 존재한다.)
- 서비스 계층에서 대부분의 비즈니스로직을 처리하는것을 트랜잭션 스크립트 패턴이라고 한다.

> 어떤 패턴이 더 좋고 나쁨을 떠나서 상황에 따라 적절히 사용하는것이 중요하다. 한 프로젝트 내에서도 두가지 패턴 모두 공존할 수 있다.

> JPA를 사용하면 트랜잭션 스크립트 패턴보다는 도메인 모델 패턴으로 많이 개발하게 된다.
