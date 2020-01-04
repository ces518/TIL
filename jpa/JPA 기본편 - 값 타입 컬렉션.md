# JPA 기본편 - 값 타입 컬렉션

#### 값 타입 컬렉션
- 값 타입 컬렉션이란 ?
    - 값 타입을 자바컬렉션에 담아서 사용하는것을 의미한다.

- Member
    - id: Long
    - favoriteFoods: Set<String>
    - addressHistory: List<Address>

- 컬렉션을 디비에 넣을때 문제가 발생
- 관계형 데이터베이스는 컬렉션 데이터를 넣을수 있는 기능을 제공하지 않음 (최신 디비는 json등을 지원함)
- 보통 별도의 테이블을 둬야한다.

- MEMBER
    - ID
- FAVORITE_FOOD
    - MEMBER_ID (PK, FK)
    - FOOD_NAME (PK)
- ADDRESS
    - MEMBER_ID (PK, FK)
    - CITY (PK)
    - STREET (PK)
    - ZIPCODE (PK)

> 값타입 이기때문에 별도의 식별자를 둬버리면 엔티티가 되어버리기 때문에 값타입은 모든 값들을 묶어서 PK로 사용한다.

```java
@Entity
public class SampleMember {
    @Id @GeneratedValue
    @Column(name = "SAMPLE_MEMBER_ID")
    private Long id;
    private String username;
    @Embedded
    private Period workPeriod;
    @Embedded
    private Address homeAddress;
    /* 값 타입을 매핑하는 애노테이션 */
    @ElementCollection
    /* 값 타입을 저장할 테이블을 매핑하는 애노테이션 */
    @CollectionTable(name = "FAVORITE_FOOD",
            joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME") // 값 타입을 저장할 컬럼명을 변경하고 싶을때 예외적으로 사용할 수 있다. (값이 하나이기떄문에 가능함, 임베디드 타입에서는 불가능)
    private Set<String> favoriteFoods = new HashSet<>();

    @ElementCollection
    @CollectionTable(name = "ADDRESS",
            joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistories = new ArrayList<>();
}
```

- 값 타입을 하나 이상 저장할때 사용한다.
- @ElementCollection, @CollectionTable 애노테이션을 사용한다.
- 데이터베이스 컬렉션은 같은 테이블에 저장할 수 없다.
    - 1:N 이기 떄문에 별도의 테이블로 만들어 내야한다.
    
> 값 타입 컬렉션은 영속성 전이 (CASCADE) + 고아 객체 제거 기능(orphanRemoval) 을 필수로 가져간다고 볼 수 있다.

#### 값 타입 컬렉션의 제약사항
- 값 타입은 엔티티와 다르게 식별자 개념이 없다.
- 값을 변경하면 추적이 어렵다.
- 값 타입 컬렉션에 변경사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션의 현재값을모두 다시 저장한다.
- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본키를 구성해야한다.
    - null x, 중복 x

> 실무에서는 사용하지 않는걸 추천..

`실무에서 사용해선 안되는 이유`
- 일단 테이블이 문제이다.
- 값이 변경되면 추적이 안된다.
    - 식별자가 없기 떄문
- 의도하지 않은대로 동작할 확률이 높다.

#### 값 타입 컬렉션 대안
- 실무에서는 상황에 따라 값 타입 컬렉션 대신 일대다 관계를 고려할것
    - 값타입을 엔티티로 승급한다고 한다.
- 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용한다.
- 영속성전이 + 고애 객체 제거를 사용해서 값 타입 컬렉션처럼 사용하면 된다.

```java
@Entity
@Table(name = "ADDRESS")
public class AddressEntity {
    @Id @GeneratedValue
    private Long id;
    
    @Embedded
    private Address address;
}
```

> 값타입 컬렉션은 매우 단순할때만 사용할것. 수정을 한다거나 값을 추적해야 하는경우는 반드시 엔티티를 사용할것


#### 정리
- 엔티티 타입
    - 식별자가 있다
    - 생명 주기를 관리
    - 공유 가능
- 값 타입
    - 식별자가 없다
    - 생명 주기를 엔티티에 의존한다
    - 공유하지 않는것이 안전하다
    - 불변 객체로 만드는것이 안전하다.

> 값 타입은 정말 값타입이라고 판단될때만 사용할것, 엔티티와 값 타입을 혼동해서, 엔티티를 값타입으로 만들어선 안된다.

* 식별자가 필요하고, 지속해서 값을 추적 및 변경해야 한다면 그것은 값 타입이 아닌 엔티티이다.
