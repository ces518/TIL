# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 엔티티 설계시 주의점

### 엔티티 설계시 주의점

#### 엔티티에는 가급적 Setter 사용 자제
- Setter가 모두 열려있따면, 변경포인트가 너무 많아져서 유지보수가 어렵다.

#### 모든 연관관계는 지연로딩으로 설정
- 즉시로딩 (EAGER)은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다.
- JPQL을 실행할 때 N+1 가 자주 발생한다.
- 실무에서 모든 연관관계는 지연로딩(LAZY)로 설정 해야한다.
- 연관된 엔티티를 함께 DB에서 조회해야 한다면, fetch join 또는 엔티티 그래프 기능을 사용한다.
- XToOne, 관계는 기본 즉시로딩이므로 지연로딩으로 모두 바꿔줄것

- JPA 가 제공하는 JPQL을 사용하면, 연관관계를 모두 무시하고 SQL로 변환된다. (아몰라, 일단 변환이다.)
- 변환된 SQL을 DB에 날려 A 엔티티만 가져오게되는데 A와 B의 연관관계가 EAGER라면 B 엔티티를 가져오기위해 쿼리가 N번 만큼 더 발생함!!

> 일단 최적화의 시작점은 모든 연간관계가 LAZY가 되어야 시작할 수 있다.

#### 컬렉션은 필드에서 초기화 하자.
- null 에서 안전하다.
- 컬렉션은 필드에서 바로 초기화 하는것이 안전하다.
- 하이버네이트는 영속화할때 컬렉션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다.
- 임의이 메소드에서 컬렉션을 잘못생성하면, 하이버네이트 내부매커니즘에 문제가 발생할 수 있다.
    - 컬렉션을 수정해선 안된다.
    - 수정할 경우 하이버네이트가 관리하는 컬렉션으로 바뀌어 버렸기때문에 제대로 동작이 하지않을 수 있다.
- 따라서 필드레벨에서 생성하는것이 가장 안전하고, 코드도 간결하다.

```java
Member member = new Member();
member.getOrders().getClass(); // java.util.ArrayList
em.persist(member);
member.getOrders().getClass(); // org.hibernate.collection.internal.PersistentBag
```

> 필드에서 초기화한다고해서, 메모리를 얼마 잡아먹지 않기때문에 베스트프렉티스인 필드 초기화 방법을 사용

#### 테이블, 컬럼명 생성 전략
- 스프링부트에서 하이버네이트 기본 매핑전략을 변경해서 실제 테이블 필드명은 다름

- 스프링부트 신규설정  (Spring Physocal Namging Strategy)
    - 1.카멜케이스 -> 언더스코어로 바꿈
    - 2.점 (.) 은 언더스코어로 바꿈
    - 3.대문자 -> 소문자

`적용 2단계`
- 1. 논리명 생성: 명시적으로 컬럼,, 테이블명을 직접 적지않으면 ImplicitNameStrategt 사용
    - spring.jpa.hibernate.naming.implicit-strategy: 테이블, 컬럼명을 명시하지 않을때 논리명 적용
- 2. 물리명 적용: 
    - spring.jpa.hibernate.naming,physical-strategy: 모든 논리명에 적용됨, 실제 테이블에 적용(username -> usernm 등으로 회사 룰로 변경이 가능)

`스프링부트 기본 설정`
```properties
org.jpa.hibernate.naming.implict-strategy: org.springframework.boot.orm.jpa.hibernate.SpringImplictNamingStrategy
org.jpa.hibernate.naming.physical-strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamgingStrategy
```

#### CASCADE
- CASCADE 옵션을 지정하면, 자식 엔티티의 라이프 사이클을 설정할 수 있음

* CascadeType.ALL 지정시 모든 라이프사이클을 부모 엔티티의 사이클을 따른다.

#### 연관관계 편의 메소드
- 양방향 연관관계 매핑시 사용한다.

```java
/* 연관관계 편의 메소드
*  연관관계 편의 메소드의 위치는 핵심 적인 엔티티에 두는것이 좋다.
* */
public void setMember (Member member) {
    this.member = member;
    member.getOrders().add(this);
}

public void addOrderItem (OrderItem orderItem) {
    orderItems.add(orderItem);
    orderItem.setOrder(this);
}

public void setDelivery (Delivery delivery) {
    this.delivery = delivery;
    delivery.setOrder(this);
}
```
