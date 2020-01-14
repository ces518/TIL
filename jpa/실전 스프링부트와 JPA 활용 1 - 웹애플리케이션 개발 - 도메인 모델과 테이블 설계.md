# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 도메인 모델과 테이블 설계

#### 도메인 모델과 테이블 설계
- 회원은 여러 상품을 주문할 수 있다.
- 한번 주문시, 여러 상품을 선택할 수 있다.
- 상품은 도서, 음반, 영화로 구분된다.

#### 엔티티 분석
- Member
    - id
    - name
    - address
    - orders
- Order
    - id
    - member
    - orderItems
    - delivery
    - orderDate
    - status
- Delivery
    - id
    - order
    - address
    - status
- OrderItem
    - id
    - item
    - order
    - orderPrice
    - count
- Item
    - id
    - name
    - price
    - stockQuantity
    - categories
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
    - id
    - name
    - items
    - parent
    - child

- 회원 (Member)
    - 이름과, 임베디드타입 (Address) 주소 와 주문 목록을 가진다.
- 주문 (Order)
    - 한번 주문시 여러 상품을 주문할 수 있으므로 주문과 주문상품은 일대다 관계
    - 주문은 상품 주문시 회원과 배송정보, 주문날짜, 주문상태를 가지고 있다, 주문상태는 enum 타입을 사용
- 주문상품 (OrderItem)
    - 주문한 상품 정보와 주문금액, 주문수량 정보를 가지고 있따.
    - 보통 OrderLine, LineItem 으로 많이 표현한다.
- 상품 (Item)
    - 이름, 가격, 재고수량을 가지고 있다.
    - 상품을 주문하면 재고수량이 줄어든다.
    - 상품의 종류로는 도서, 음반, 영화가 있는데 각각은 사용하는 속성이 조금씩 다르다.
- 배송 (Delivery)
    - 주문시 하나의 배송 정보를 생성한다.
    - 주문과 배송은 일대일 관계다.
- 카테고리 (Category)
    - 상품과 다대다 관계를 맺는다.
    - parent, child 로 부모,자식 카테고리를 연결한다.
- 주소 (Address)
    - 값타입 이다.

> 회원을 주문을 하기 때문에, 회원이 주문리스트를 가지는것은 잘 설계한것 같지만, 실무에서는 회원이 주문을 참조하지 않고, 주문이 회원을 참조하는것으로 충분하다.
- 회원이 주도하는 것 이아닌, 주문을 생성할때, 회원이 필요하다고 생각하는 것이 맞다.

#### 회원 테이블 분석
- MEMBER
    - MEMBER_ID
    - NAME
    - CITY
    - STREET
    - ZIPCODE
- ORDERS
    - ORDER_ID
    - MEMBER_ID (FK)
    - DELIVERY_ID (FK)
    - ORDER_DATE
    - STATUS
- ORDER_ITEM
    - ORDER_ITEM_ID
    - ORDER_ID (FK)
    - ITEM_ID (FK)
    - ORDERPRICE
    - COUNT
- DELIVERY
    - DELIVERY_ID
    - STATUS
    - CITY
    - STREET
    - ZIPCODE
- ITEM
    - ITEM_ID
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
- CATEGORY
    - CATEGORY_ID (FK)
    - PARENT_ID (FK)
    - NAME
- CATEGORY_ITEM
    - CATGORY_ID
    - ITEM_ID (FK)

> Item의 상속관계는 싱글테이블 전략을 사용

#### 연관관계 매핑 분석
- 회원과 주문
    - 일대다, 다대일의 양방향 관계
    - 연관관계의 주인을 정해야하는데, 외래 키가 있는 **주문** 을 연관관계의 주인으로 정하는것이 좋다. (FK를 가지고 있는 다 쪽이 주인)
    - Order.member 를 ORDERS.MEMBER_ID 와 매핑한다.
- 주문상품과 주문
    - 다대일 양방향 관계
    - 외래키가 주문상품에 있으므로, 주문상품이 연관관계의 주인이다.
    - OrderItem.order 를 ORDER_ITEM.ORDER_ID 와 매핑한다.
- 주문상품과 상품
    - 다대일 단방향 관계
    - OrderItem.item 을 ORDER_ITEM.ITEM_ID 외래 키와 매핑한다.
- 주문과 배송
    - 일대일 단방향 관계
    - Order.delivery 를 ORDERS.DELIVERY_ID 외래 키와 매핑한다.
- 카테고리와 상품
    - @ManyToMany를 사용해서 매핑한다.
        - 실무에서는 사용하지 말것

> **외래 키가 있는곳을 연관관계의 주인으로 정하라.**
> 연관관계의 주인은 외래키를 누가 관리하냐의 단순한 문제일뿐, 비즈니스상 우위에 있다고 해서 주인으로 정해선 안된다..
> 자동차와 바퀴가 일대다 관계에서 외래키가 있는 바퀴를 연관관계의 주인으로 정하면된다.
> 자동차를 연관관계의 주인으로 정하면, 자동차가 관리하지 않는 바퀴 테이블의 외래키 값이 업데이트 되므로, 관리 및 유지보수가 어렵다.
> 추가적인 쿼리가 발생하는 성능 문제도 있다.
