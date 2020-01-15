# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 엔티티클래스 개발 2

#### 참고사항
`Getter 와 Setter`
- 이론적으로 Getter, Setter 모두 제공하지않고, 꼭 필요한 별도의 메소드만 제공하는것이 이상적이다.
- 하지만 실무에서는 조회 할일이 많으며, Getter 만으로는 문제가 발생하지 않는다.
- Setter의 경우에는 문제가 다르다.
- **Setter가 존재한다는것은 변경될 여지를 열어두는것이기 때문에, 엔티티가 어디서 변경되었는지 추적하기가 상당히 힘들다**.
- Setter대신 **변경지점이 명확하도록 변경을 위한 비즈니스 메소드를 별도로 제공**해야한다.
- 애플리케이션이 복잡해질수록 유지보수가 힘들다.

`@ManyToMany`
- 객체는 다대다 컬렉션 참조가 가능하지만, 관계형 DB에서는 불가능하다.
- 중간 테이블로 매핑을 해주어야한다.
- 객체와 관계형 DB의 차이점을 해소하기 위함
- 실무에서는 다대다 매핑은 사용하지 말것.
- 컬럼 추가등 커스터마이징이 불가능하다.
- 운영하기가 힘들다.
```java
@Entity
@Getter @Setter
public class Category {

    @Id @GeneratedValue
    @Column(name = "category_id")
    private Long id;

    private String name;

    // 객체는 다대다 컬렉션 참조가 가능하지만, 관계형 DB에서는 불가능하다.
    // 중간 테이블로 매핑을 해주어야한다.
    // 객체와 관계형 DB의 차이점을 해소하기 위함
    // 실무에서는 다대다 매핑은 사용하지 말것.
    // 컬럼 추가등 커스터마이징이 불가능하다.
    @ManyToMany
    @JoinTable(name = "category_item",
        joinColumns = @JoinColumn(name = "category_id"),
        inverseJoinColumns = @JoinColumn(name = "item_id") // 반대편인 아이템에 해당하는 FK 컬럼명 설정
    )
    private List<Item> items = new ArrayList<>();

    // 계층형 구조의 부모 매핑
    // 다대일 양방향 매핑
    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Category parent;

    // 계층형 구조의 자식 매핑
    // 일대다 양방향 매핑
    @OneToMany(mappedBy = "parent")
    private List<Category> children = new ArrayList<>();
}
```


`값타입`
- 값타입은 변경이 되어서는 안된다.
- 가장 좋은 설계는 생성될때만 값이 세팅이 되도록 하는것이다.
- Setter를 제공하지 말 것
- public 생성자보다는 protected 생성자를 두는것이 좋다.
- 기본생성자가 반드시 필요한 이유는 JPA 구현라이브러리가 객체를 생성할때 리플렉션같은 기술을 사용할 수 있도록 지원해야 하기 때문이다.
```java
@Embeddable
@Getter
public class Address {

    private String city;
    private String street;
    private String zipcode;

    // JPA 에서는 기본생성자가 반드시 필요하다.
    // public 생성자가 존재한다면 불변객체로 생성하기가 힘들 경우가많다.
    // 따라서 JPA 에서는 protected 생성자까지 허용한다.
    protected Address() {
    }

    public Address (String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}

```

> 테이블 설계시 정합성이 중요할 경우 FK를 사용하고, 정합성보다는 유연하게 잘 서비스 되는것이 중요하다면, 인덱스만 거는것을 고려 하자.
