# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 주문 기능 테스트

#### 주문 기능 테스트
`테스트 요구사항`
- 상품 주문이 성공해야 한다.
- 상품을 주문할 때 재고수량을 초과해선 안된다.
- 주문 취소가 성공해야 한다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class OrderServiceTest {

    @Autowired EntityManager em;

    @Autowired OrderService orderService;

    @Autowired OrderRepository orderRepository;

    @Test
    public void 상품주문 () throws Exception {
        //== given ==//
        Member member = createMember();
        Item book = createBook("JPA", 10000, 10);

        //== when ==//

        // 회원이 책 2권 주문 테스트
        int orderCount = 2;
        Long orderId = orderService.order(member.getId(), book.getId(), orderCount);

        //== then ==//
        Order getOrder = orderRepository.findById(orderId);
        assertEquals("상품 주문시 주문 상태는 ORDER", OrderStatus.ORDER, getOrder.getStatus());
        assertEquals("주문한 상품 종류 수가 정확해야 한다.", 1, getOrder.getOrderItems().size());
        assertEquals("주문 가격은 가격 * 수량이다.", 10000 * orderCount, getOrder.getTotalPrice());
        assertEquals("주문 수량만큼 재고가 줄어야 한다.", 8, book.getStockQuantity());
    }

    @Test(expected = NotEnoughException.class)
    public void 재고수량_초과 () throws Exception {
        //== given ==//
        Member member = createMember();
        Item book = createBook("JPA", 10000, 10);

        int orderCount = 11;

        //== when ==//
        orderService.order(member.getId(), book.getId(), orderCount);

        //== then ==//
        fail("재고 수량 부족 예외가 발생해야 한다.");
    }

    @Test
    public void 주문취소 () throws Exception {
        //== given ==//
        Member member = createMember();
        Item book = createBook("JPA", 10000, 10);

        int orderCount = 2;
        //이번에는 취소에 대한 테스트 이기 때문에 주문까지 given에 존재한다.
        Long orderId = orderService.order(member.getId(), book.getId(), orderCount);

        //== when ==//
        //테스트하고 싶은 항목이 when 절에 위치
        orderService.cancelOrder(orderId);

        //== then ==//
        Order getOrder = orderRepository.findById(orderId);
        assertEquals("주문 취소시 상태는 CANCEL 이 되어야 한다.", OrderStatus.CANCEL, getOrder.getStatus());
        assertEquals("주문이 취소된 상품은 그만큼 재고가 증가해야 한다.", 10, book.getStockQuantity());
    }

    // command + option + p 클릭시 파라메터 변수로 뺄수 있음
    private Item createBook(String name, int price, int stockQuantity) {
        // 책 세팅
        Item book = new Book();
        book.setName(name);
        book.setPrice(price);
        book.setStockQuantity(stockQuantity);
        em.persist(book);
        return book;
    }

    private Member createMember() {
        // 회원 세팅
        Member member = new Member();
        member.setName("멤버1");
        member.setAddress(new Address("대전" ,"어딘가", "1234-1234"));
        em.persist(member);
        return member;
    }
}
```

> 실무에서는 위의 테스트보다 훨씬 더 꼼꼼하게 진행해야 한다.

> 사실 위의 테스트 코드는 실무에 가깝다기 보단 SpringBoot 와 JPA가 엮여서 잘 동작하는지 확인하는 통합테스트에 가깝다.

> 가장 좋은 테스트는 단위테스트를 의미있게 잘 작성하는 것이다. 현재 비즈니스로직이 엔티티에 위치하고 있기때문에 엔티티의 메소드를 테스트하는 단위테스트도 필요하다.
