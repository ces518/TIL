# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 주문 검색 기능 개발

#### 주문 검색 기능 개발
- JPA에서 동적 쿼리를 어떻게 처리해야 하는가 ?

`검색 조건 파라미터 OrderSearch`
```java
@Getter @Setter
public class OrderSearch {

    private String memberName; // 회원명
    private OrderStatus orderStatus; // 주문상태: [ORDER, CANCEL]
}
```

`검색 조건기능 개발`
```java
  /**
     * 주문 검색기능 추후 구현
     */
    public List<Order> findAll (OrderSearch orderSearch) {


        // 아래 쿼리는 값이 다 있다는 가정..
        List<Order> orders = em.createQuery("select o from Order o join o.member m " +
                "where o.status = :status " +
                "and m.name like :name", Order.class)
                .setParameter("status", orderSearch.getOrderStatus())
                .setParameter("name", orderSearch.getMemberName())
                .setMaxResults(1000) // 최대 1000건
                .getResultList();

        //== Criteria 사용 방법 ==//
        // JPA 표준
        // 실무에서 사용하기 어렵다.
        // 단점: 유지보수성이 거의 0에 가깝다.
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Order> cq = cb.createQuery(Order.class);
        Root<Order> o = cq.from(Order.class);
        Join<Object, Object> m = o.join("member", JoinType.INNER);

        List<Predicate> criteria = new ArrayList<>();

        if (orderSearch.getOrderStatus() != null) {
            Predicate status = cb.equal(o.get("status"), orderSearch.getOrderStatus());
            criteria.add(status);
        }
        if (StringUtils.hasLength(orderSearch.getMemberName())) {
            Predicate name = cb.like(m.get("name"), "%" + orderSearch.getMemberName() + "%");
            criteria.add(name);
        }
        cq.where(criteria.toArray(new Predicate[criteria.size()]));
        orders = em.createQuery(cq)
                .setMaxResults(1000)
                .getResultList();
        //== ==//
        return orders;
    }
```

- JPQL을 사용한 동적쿼리는 결국 문자열이기 때문에 동적쿼리를 처리하기 상당히 어렵다.
- JPA 표준스펙에서 지원하는 Criteria를 사용하면 그나마 좀 나아지지만, 실무에서 사용하기 어렵고 유지보수성이 거의 0에 가깝다.
- QueryDSL 이라는 라이브러리를 사용해서 처리하는것을 추천함.
