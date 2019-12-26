# JPA 기본편 - 실전예제 4. 상속관계 매핑

#### 요구사항 추가
- 상품의 종류는 음반, 도서, 영화가 있고 이후 더 확장될 수 있다.
- 모든 데이터는 등록일과 수정일이 필수이다.

#### 도메인 모델 - ERD
- MEMBER
    - MEMBER_ID (PK)
    - NAME
    - CITY
    - STREET
    - ZIPCODE
- ORDERS
    - ORDER_ID (PK)
    - MEMBER_ID (FK)
    - DELIVERY_ID (FK)
    - ORDERDATE
    - STATUS
- ORDER_ITEM
    - ORDER_ITEM_ID (PK)
    - ORDER_ID (FK)
    - ITEM_ID (FK)
    - ORDERPIRCE
    - COUNT
- CATEGORY
    - CATEGORY_ID (PK)
    - PARENT_ID (FK)
    - NAME
- CATEGORY_ITEM
    - CATEGORY_ID (FK)
    - ITEM_ID (FK)
- DELIVERY
    - DELIVERY_ID (PK)
    - CITY
    - STREET
    - ZIPCODE
    - STATUS
- ITEM
    - ITEM_ID (PK)
    - NAME
    - PRICE
    - STOCKQUANTITY
    - DTYPE
    - ARTIST
    - ETC
    - AUTHOR
    - ISBN
    - DIRECTOR
    - ACTOR


#### 도메인 모델 - 엔티티
- Member
    - id: Long
    - name: String
    - city: String
    - street: String
    - zipcode: String
    - orders: List
- Orders
    - id: Long
    - member: Member
    - orderItems: List
    - delivery: Delivery
    - orderDate: Date
    - status: OrderStatus
- OrderItem
    - id: Long
    - order: Order
    - item: Item
    - orderPrice: int
    - count: int
- Category
    - id: Long
    - parent: Category
    - name: String
    - items: List
    - child: List
- Delivery
    - id: Long
    - city: String
    - street: String
    - zipcode: String
    - status: DeliveryStatus
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

> 테이블은 싱글 테이블로 설계하고, 상속관계 매핑을 사용한 모델링

#### 정리
- 처음에는 객체지향적으로 설계 및 모델링해서 운영을 해나가다 기점이 오게됨
- 하루에 100만건, 데이터가 억단위이상으로 넘어가게되면 테이블을 단순하게 유지해야한다.
- 애플리케이션이 점점 커질수록 복잡도가 증가하기 때문에 테이블은 최대한 단순하게 유지해야한다.
- 그런 상황이 왔을때 적절한 트레이드오프를 해서 애플리케이션을 유지해 나가는것이 중요함
