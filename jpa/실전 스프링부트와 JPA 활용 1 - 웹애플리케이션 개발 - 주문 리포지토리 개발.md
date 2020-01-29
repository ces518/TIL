# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 주문 리포지토리 개발

#### 주문 리포지토리 개발
- 주문기능
- 단건조회
- 목록 검색기능
    - 검색기능은 동적쿼리가 들어가서 복잡하기 때문에 추후 구현

```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {
    private final EntityManager em;

    /**
     * 주문 기능
     * @param order
     */
    public void save (Order order) {
        em.persist(order);
    }

    /**
     * 주문 단건조회
     * @param orderId
     * @return
     */
    public Order findById (Long orderId) {
        return em.find(Order.class, orderId);
    }

    /**
     * 주문 검색기능 추후 구현
     */
    /*
    public List<Order> findAll (OrderSearch orderSearch) {
        return null;
    }
    */
}
```
