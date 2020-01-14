# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 엔티티클래스 개발 1

#### 엔티티 클래스개발 
- Getter Setter를 모두 열고, 최대한 단순하게 설계한다.
- 실무에서는 가급적 Getter만 열어두고, Setter는 꼭 필요한 경우에서만 사용할것

> 1:1 관계에서 FK는 주로 ACCESS 하는 쪽에 두는 것을 추천한다.


```java
@Entity
@Getter @Setter
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

    @Embedded
    private Address address;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}
@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {

    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    /*
    @Temporal(TemporalType.TIMESTAMP)
    private Date date;
     */
    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status; // 주문상태: ORDER, CANCEL
}
public enum OrderStatus {
    ORDER, CANCEL
}
@Entity
@Getter @Setter
public class Delivery {

    @Id @GeneratedValue
    @Column(name = "delivery_id")
    private Long id;

    @OneToOne(mappedBy = "delivery")
    private Order order;

    @Embedded
    private Address address;

    @Enumerated(EnumType.STRING)
    private DeliveryStatus status; // 배송상태: READY, COMP
}
public enum DeliveryStatus {
    READY, COMP
}
@Entity
@Getter @Setter
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;

    @ManyToOne
    @JoinColumn(name ="item_id")
    private Item item;

    private int orderPrice; // 주문가격 (주문당시 가격)

    private int count; // 수량
}
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype") // 구분값 컬럼 기본값은 DTYPE
@Getter @Setter
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;

    private int price;

    private int stockQuantity;
}
@Entity
@DiscriminatorValue("B") // 구분값 벨류 설정
@Getter @Setter
public class Book extends Item {

    private String author;
    private String isbn;
}
@Entity
@DiscriminatorValue("A")
@Getter @Setter
public class Album extends Item {

    private String artist;
    private String etc;
}
@Entity
@DiscriminatorValue("M")
@Getter @Setter
public class Movie extends Item {

    private String director;
}
@Embeddable
@Getter
public class Address {

    private String city;
    private String street;
    private String zipcode;
}
```

- Item - Book, Album, Movie 상속 관계를 표현하기위해 Item class는 추상클래스로 정의
- @Inheritance
    - 상속관계 매핑 전략 설정
    - 기본값은 JOINED 전략 (가장 정규화된 설계)
- @DiscriminatorColumn
    - 싱글테이블 전략 사용시, 구분값 컬럼명 지정
    - 기본값은 DTYPE
- @DiscriminatorValue
    - 구분값 컬럼의 값 지정
    - 기본값은 엔티티클래스 명

- 임베디드타입 (값타입) 활용
    - Address
    - 주소관련 값들을 임베디드 타입을 활용해서 재활용

- ENUM 타입 사용시 주의할것
    - @Enumerated
    - EnumType.STRING 반드시 지정
    - EnumType.ORDINAL 지정시, 순서값이 들어가게 됨
    - 만약 순서가 변경된다면 장애 발생

- JAVA8, 하이버네이트 최신버전 사용시 LocalDate, LocalDateTime 사용
    - 이전 버전에서는 Date 타입과 @Temporal 애노테이션을 사용
    - LocalDate, LocalDateTime 사용시 애노테이션 생략
