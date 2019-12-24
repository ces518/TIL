# JPA 기본편 - 실전 예제 3. 다양한 연관관계 매핑

#### 배송, 카테고리 추가 - 엔티티
- 주문과 배송은 1:1 관계이다 @OneToOne
- 상품과 카테고리는 N:M 관계이다 @ManyToMany

#### 배송, 카테고리추가 - ERD

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
- ITEM
    - ITEM_ID (PK)
    - NAME
    - PRICE
    - STOCKQUANTITY
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

> DELIVERY , ORDERS 의 관계에서 주테이블인 ORDERS에 FK를 주는 방식을 선택

#### 배송, 카테고리 추가 - 엔티티 상세


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
- Item
    - id: Long
    - name: String
    - price: int
    - stockQuantity: int
    - categories: List
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

> 객체는 다대다 관계가 성립하기 때문에 Category와 Item사이의 중간 객체가 없다, 서로 컬렉션으로 참조가 가능하다.


#### N:M 관계는 1:N, N:1 로...
- 테이블의 N:M 관계는 중간 테이블을 이용해서 1:N, N:1 로 풀어낼것
- 실무에서 중간테이블은 단순하지 않다.
- @JoinTable은 필드 추가를 할 수 없으며  및 엔티티 테이블 불일치.

#### @JoinColumn
- 외래 키를 매핑할 때 사용한다.

`속성`
- name
    - 매핑할 외래키 명
    - 기본값: 필드명_참조테이블의기본키컬럼명
- referencedColumnName
    - 외래 키가 참조하는 대상 테이블의 컬럼명
    - 참조하는 테이블의 기본키 컬럼명
- foreignKey(DDL)
    - 외래 키 제약조건을 직접 지정할 수 있다.
    - 이 속성은 테이블 생성시에만 사용한다.
- unique, nullable insertable, updatable, columnDefinition, table
    - @Column의 속성과 동일하다


#### @ManyToOne
- 다대일 관계 매핑

- optional
    - false 설정시 연관된 에니티가 항상 있어야한다
    - 기본값: true
- fetch
    - 글로벌 패치 전략을 설정한다.
    - XToOne: EAGER
    - XToMany: LAZY
- cascade
    - 영속성 전이 기능을 사용한다.
- targetEntity
    - 연관된 엔티티 타입의 정보를 설정한다.
    - 이 기능은 거의 사용하지 않음
    - 컬렉션을 사용해도 제네릭으로 정보를 알 수 있음

> 다대일을 사용하면 무조건 연관관계의 주인이 되어야한다. (파훼법은 있음)

#### @OneToMany
- 일대다 관계 매핑

- mappedBy
    - 연관관계 주인 필드를 선택한다.
- fetch
    - 글로벌 패치 전략을 설정한다.
    - XToOne: EAGER
    - XToMany: LAZY
- cascade
    - 영속성 전이 기능을 사용한다.
- targetEntity
    - 연관된 엔티티 타입의 정보를 설정한다.
    - 이 기능은 거의 사용하지 않음
    - 컬렉션을 사용해도 제네릭으로 정보를 알 수 있음
