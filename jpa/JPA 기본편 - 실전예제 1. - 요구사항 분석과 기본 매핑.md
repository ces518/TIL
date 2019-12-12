# JPA 기본편 - 실전예제 1. - 요구사항 분석과 기본 매핑

#### 요구사항
- 회원은 상품을 주문할 수 있다.
- 주문시 여러 종류의 상품을 선택할 수 있다.

#### 기능 목록
- 회원 기능
    - 회원등록
    - 회원조회
- 상품 기능
    - 상품등록
    - 상품수정
    - 상품조회
- 주문 기능
    - 상품주문
    - 주문내역조회
    - 주문취소

#### 도메인 모델 분석
- 회원과 주문의 관계
    - 회원을 여러번 주문할 수 있다.
- 주문과 상품의 관계
    - 주문할 때 여러 상품을 선택할 수 있다.
    - **주문 상품** 이라는 모델을 만들어, 다대다 관계를 일대다, 다대일 관계로 풀어 냄


#### 테이블 설계
- MEMBER
    - MEMBER_ID
    - NAME
    - CITY
    - STREET
    - ZIPCODE

- ORDERS
    - ORDER_ID
    - MEMBER_ID(FK)
    - ORDERDATE
    - STATUS

- ORDER_ITEM
    - ORDER_ITEM_ID
    - ORDER_ID(FK)
    - ITEM_ID(FK)
    - ORDERPRICE
    - COUNT

- ITEM
    - ITEM_ID
    - NAME
    - PRICE
    - STOCKQUANTITY


#### 엔티티 설계와 매핑
- MEMBER
    - id: Long
    - name: String
    - city: String
    - street: String
    - zipcode: String

- ORDERS
    - id: Long
    - memberId: Long
    - orderDate: Date
    - stauts: OrderStatus

- ORDER_ITEM
    - id: Long
    - orderId: Long
    - itemId: Long
    - orderPrice: int
    - count: int

- ITEM
    - id: Long
    - name: String
    - price: int
    - stockQuantity: int


#### 데이터 중심 엔티티 설계
```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;

    private String city;

    private String street;

    private String zipcode;
}

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
@Table(name = "ORDERS")
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @Column(name = "MEMBER_ID")
    private Long memberId;
    
    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;
}

public enum OrderStatus {
    ORDER, CANCEL;
}

@Entity
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;

    @Column(name = "ORDER_ID")
    private Long orderId;

    @Column(name = "ITEM_ID")
    private Long itemId;

    private int orderPrice;

    private int count;
}
```

> 처음에는 보통 이런식으로 엔티티 설계가 나온다. 테이블과 동일하게 설계한 것이다.

* tip
    - setter 생성시 고민할 필요가 있음.
    - setter를 막 만들면 좋지 않다.
    - 가급적 생성자에서 값을 세팅을 다하고, setter 사용을 최소화 하는것이 코드추적시에 좋음.
    - 유지보수성이 떨어질 수 있다.
    - 스프링 부트는 매핑시 java camel-case 를 DB underscore case 를 기본 설정으로 한다.

기존의 설계 방식은 DB중심의 설계이다.

#### 데이터 중심 설계의 문제점
- 현재 방식은 객체 설계를 테이블 설게에 맞춘 방식
- 테이블 외래키를 객체에 그대로 가져옴
- 객체 그래프 탐색이 불가능함
- 참조가 없기 때문에 UML도 잘못됨
