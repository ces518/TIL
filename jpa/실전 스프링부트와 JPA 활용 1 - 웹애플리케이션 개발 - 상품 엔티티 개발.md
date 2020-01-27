# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 상품 엔티티 개발

#### 상품 엔티티 개발
`구현 기능`
- 상품 등록
- 상품 목록 조회
- 숭품 수정

`순서`
- 엔티티 개발 (비즈니스 로직 개발)
- 상품 리포지토리 개발
- 상품 서비스개발, 기능 테스트

#### 상품 엔티티
```java
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

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();

    /**
     * 도메인주도 설계시
     * 엔티티가 해결할 수 있는 비즈니스 로직은 엔티티가 가지고 있는것이 좋다.
     * 데이터가 가지고 있는쪽에서 비즈니스 로직을 두는것이 좋음
     * 객체지향적인 설계
     * 응집도 증가
     */

    /**
     * 재고 증가
     * @param quantity
     */
    public void addStock (int quantity) {
        this.stockQuantity += quantity;
    }

    /**
     * 재고 감소
     * @param quantity
     */
    public void removeStock (int quantity) {
        int resultStock = this.stockQuantity - quantity;
        if (resultStock < 0) {
            throw new NotEnoughException("need more stock");
        }
        this.stockQuantity = resultStock;
    }
}
```

> 핵심 비즈니스 로직은 데이터를 가지고 있는 엔티티가 가지고 있는것이 좋다 > 객체지향적인설계, 응집도 증가
