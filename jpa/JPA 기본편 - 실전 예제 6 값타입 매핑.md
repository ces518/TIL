# JPA 기본편 - 실전 예제 6 값타입 매핑

#### 값타입 매핑
- Member
    - id: Long
    - name: String
    - address: Address
    - orders: List

- Order
    - id: Long
    - member: Member
    - orderItems: List
    - delivery: Delivery
    - orderDate: Date
    - status: OrderStatus

- OrderItem
    - id: Long
    - item: Item
    - order: Order
    - orderPrice: int
    - count: int

- Item
    - id: Long
    - name: String
    - price: int
    - stockQuantity: int
    - categories: List

- Album
    - artist
    - etc
- Book
    - author
    - isbn
- Movie
    - director
    - actor

- Category
    - id: Long
    - name: String
    - items: List
    - parent: Category
    - child: List

- Delivery
    - id: Long
    - order: Order
    - address: Address
    - status: DeliveryStatus

[Value Type]
- Address
    - city: String
    - street: String
    - zipcode: String

> Address 를 값타입으로 만들어서 적용한다.


```java
@Embeddable
public class Address {

    @Column(length = 10)
    private String city;

    @Column(length = 20)
    private String street;

    @Column(length = 5)
    private String zipcode;

    // 의미있는 비즈니스 메서드를 만들수 있다.
    public String fullAddress () {
        return getCity() + getCity() + " " + getZipcode();
    }

    public String getCity() {
        return city;
    }

    public String getStreet() {
        return street;
    }

    public String getZipcode() {
        return zipcode;
    }

    // getter로 호출해야 프록시객체일때 문제가 발생하지 않는다.
    // 대부분의 코드를 메소드를 호출하는 방식으로 짜는것이 안전하다.
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(getCity(), address.getCity()) &&
                Objects.equals(getStreet(), address.getStreet()) &&
                Objects.equals(getZipcode(), address.getZipcode());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getCity(), getStreet(), getZipcode());
    }
}
```


#### 값타입을 사용했을때의 장점
- 한군데 에서만 쓰이더라도 상당히 의미가 있다.
- 의미있는 비즈니스 메소드를 만들 수 있다.
- 값타입을 사용하면 validationRule도 공통으로 적용이 가능하며, 관리가 쉽다.
- 객체지향적으로 더 사용할 수 있다.
