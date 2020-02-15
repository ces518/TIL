# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_조회API_V4

#### JPA 에서 바로 DTO로 조회하기
- 엔티티에서 DTO로 변환하는것이 아닌 JPA에서 바로 DTO로 조회하기
```java
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4 () {
    return orderRepository.findOrderDtos();
}
@Data
public class OrderSimpleQueryDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    // 식별자
    public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name; // Lazy 초기화 (쿼리 발생)
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address; // Lazy 초기화 (쿼리 발생)
    }
}
// 엔티티나 값타입만 반환이 가능하다.
// DTO로 반환하려면 new Operation을 사용해야한다.
// 재사용성이 떨어진다.
// DTO로 조회했기때문에 비즈니스 로직 등을 활용할 수 없다.
public List<OrderSimpleQueryDto> findOrderDtos() {
    // JPQL에서 엔티티인 o 를 넘기면 엔티티가 아닌 식별자가 넘어가기때문에 필드를 하나하나 지정해 주어야한다..
    return em.createQuery(
            "select new jpabook.jpashop.repositories.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address) from Order o " +
                    "join o.member m " +
                    "join o.delivery d", OrderSimpleQueryDto.class)
            .getResultList();
}
```
- JPA Repository에서는 엔티티나 값타입만 반환이 가능하다.
- new 명령어를 사용해 JPQL의 결과를 DTO로 즉시 변환하여 사용할 수 있다.

`주의점`
- new 명령어를 사용할때 JPQL에서 엔티티의 알리아스를 넘기면 엔티티가 아닌 식별자가 넘어가기 때문에, select절에서 필드를 하나 하나 지정해 주어야한다.

`장/단점`
- DB에서 애플리케이션 용량 최적화
    - 생각보다 미비
- 리포지토리 재사용성이 떨어짐
    - API스펙에 맞춰서 되어있기때문에 해당 API에서만 활용이 가능함
    - 객체 그래프 조회 등이 부족함
    - 논리적으로 계층이 깨져있는것이다.
    - 리포지토리가 화면에 의존하고 있다.

> 필드를 몇개 더 줄인다고해서 성능차이에 큰 영향을 주지 않는다. 대부분은 인덱스 문제이다..

#### 정리
`쿼리 방식 선택 순서`
- 1.우선 엔티티를 DTO로 변환하는 방법을 선택
- 2.필요시 페치조인으로 성능을 최적화 한다 -> 대부분의 성능 이슈가 해결된다.
- 3.그래도 안된다면 대부분 DTO로 직접 조회하는 방법을 사용한다.
- 4.최후의 방법은 JPA가 제공하는 네이티브SQL이나 JdbcTemplate으로 SQL을 직접 사용한다.

- 리포지토리는 엔티티를 조회하는 용도로만 사용해야한다.
- DTO조회용도(화면에 종속적)라면 따로 분리하는것이 좋다. -> 유지보수성이 좋아진다.

> V3와 V4는 우열을 가리기 힘들다.. 둘다 장단이 있기 때문에 필요에 따라서 골라 사용해야한다.
