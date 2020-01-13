# JPA 기본편 - 임베디드 타입(복합값 타입)

#### 임베디드 타입
- 새로운 값 타입을 직접 정의할 수 있다.
- JPA는 임베디드 타입이라고 한다.
- 주로 기본 값 타입을 모아서 복합 값 타입 이라고도 한다.
- int, String 과 같은 값 타입

`예제`
- 회원 엔티티는 이름, 근무 시작일, 근무 종료일, 주소 도시, 주소 번지, 주소 우편번호를 가진다.

- Member
    - id
    - name
    - startDate
    - endDate
    - city
    - street
    - zipCode

- 회원 엔티티는 이름, 근무기간 집 주소를 가진다.
    - 추상화 된 정의
    - 다음과 같이 묶어서 표현할 수 있는것을 임베디드 타입이라고 한다.

- Member
    - id
    - name
    - period
    - address

- Period
    - startDate
    - endDate

- Address
    - city
    - street
    - zipcode


```java
@Embeddable
public class Period {
    private LocalDateTime startDate;
    private LocalDateTime endDate;

    public boolean isWork () {
        LocalDateTime nowDate = LocalDateTime.now();
        return startDate.isBefore(nowDate) && endDate.isAfter(nowDate);
    }
}
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipCode;
}
@Entity
public class SampleMember {

    @Id @GeneratedValue
    @Column(name = "SAMPLE_MEMBER_ID")
    private Long id;

    private String username;

    // @Embeddable 과 @Embedded 를 하나를 생략할 수 있지만 둘다 명시하는것을 추천한다.
    @Embedded
    private Period workPeriod;

    @Embedded
    private Address homeAddress;
}
```

#### 임베디드 타입 사용법
- @Embeddable
    - 값 타입을 정의하는 곳에 사용한다.

- @Embbedded
    - 값 타입을 사용하는 곳에 사용한다.

- 기본 생성자는 필수


##### 임베디드 타입의 장점
- 재사용 성 증가
- 높은 응집도
- Period.isWork()와 같은 해당 값 타입만 사용하는 의미있는 메소드를 만들 수 있다.
- 임베디드 타입을 포함한 모든 값 타입은 값 타입을 소유한 엔티티에 생명주기를 의존한다.

> 객체지향 적인 설계가 가능해진다.

#### 임베디드 타입과 테이블 매핑
- Membmer
    - id
    - name
    - Period
        - startDate
        - endDate
    - Addreess
        - city
        - street
        - zipCode

> 객체는 데이터뿐만이 아니라 행위 (Method)를 가지고 있기 때문에 임베디드 타입을 사용 했을 때 얻을 수 있는 이점이 존재한다.

- 임베디드 타입은 엔티티의 값일 뿐이다.
- 임베디드 타입을 사용하기 전 후에 **매핑하는 테이블은 같다.**
- 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는것이 가능하다.
- 잘 설계한 ORM 애플리케이션은 테이블의 수보다 클래스의 수가 많다.

> 실무에서 임베디드 타입을 엄청나게 많이 사용하지는 않는다. 하지만 공통화를 하면서 얻는 장점이 존재한다.

#### 임베디드 타입과 연관관계
- 임베디드타입은 엔티티를 가질수 있음

- Member
    - PhoneNumber
        - PhoneEntity
    
#### @AttributeOverride: 속성 재정의
- 한 엔티티 에서 같은 값을 사용한다면 ?
- 컬럼 명이 중복된다.
- @AttributeOverrides, @AttributeOverride를 사용해서 컬럼 명 속성을 재정의 할 수 있다.

```java
@Entity
public class SampleMember {

    @Id @GeneratedValue
    @Column(name = "SAMPLE_MEMBER_ID")
    private Long id;

    private String username;

    // @Embeddable 과 @Embedded 를 하나를 생략할 수 있지만 둘다 명시하는것을 추천한다.
    @Embedded
    private Period workPeriod;

    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
            @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
            @AttributeOverride(name = "zipCode", column = @Column(name = "WORK_ZIPCODE"))
    })
    private Address workAddress;
```

> Address 임베디드 타입을 활용해 homeAddress, workAddress 를 모두 매핑하고 싶을때 그대로 사용한다면 컬럼명이 중복되어 매핑이 되지 않는다. 이때 @AttributeOverrides, @AttributeOverride를 활용해 풀어낼 수 있다.

#### 임베디드 타입 과 null
- 임베디드 타입의 값이 null 이라면 매핑한 컬럼 값은 모두 null이다.
