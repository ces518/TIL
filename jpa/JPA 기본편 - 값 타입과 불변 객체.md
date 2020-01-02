# JPA 기본편 - 값 타입과 불변 객체
- 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다.
- 따라서 값 타입은 단순하고 안전하게 다를 수 있어야 한다.

> 개발할때 엔티티에 대해서는 신경을 많이 쓰지만 값을 복사하는 것에 대해 별로 신경을 쓰지 않는다..

#### 값 타입 공유 참조
- 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험하다.
- 부작용 (site Effect)가 발생한다.

```java
/* 값 타입은 엔티티간에 공유가 된다. */
Address address = new Address("city", "street", "10000");
SampleMember member = new SampleMember();
member.setUsername("Member1");
member.setHomeAddress(address);
em.persist(member);

SampleMember member2 = new SampleMember();
member2.setUsername("Member2");
member2.setHomeAddress(address);
em.persist(member2);

// 첫번째 회원의 City만 NewCity로 변경하고자 한다.
member.getHomeAddress().setCity("NewCity");
// 하지만 업데이트 쿼리가 2번 나간다.
// 참조를 공유하기 때문에 2번째 회원까지 수정이 된다..
```

> 임베디드 타입은 여러 엔티티에서 공유할 수 있다. 하지만 값 타입은 부작용이 발생해서는 안된다.

#### 값 타입 복사
- 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험하다.
- 대신 값을 복사해서 사용해야한다.
```java
/* 1. 값 타입은 엔티티간에 공유가 된다. */
Address address = new Address("city", "street", "10000");
SampleMember member = new SampleMember();
member.setUsername("Member1");
member.setHomeAddress(address);
em.persist(member);

/* 5. 사이드 이펙트를 방지하기 위해 값타입은 copy해서 사용해야 한다.*/
Address copyAddress = new Address(address.getCity(), address.getStreet(), address.getZipCode());

SampleMember member2 = new SampleMember();
member2.setUsername("Member2");
member2.setHomeAddress(copyAddress);
em.persist(member2);

// 2. 첫번째 회원의 City만 NewCity로 변경하고자 한다.
member.getHomeAddress().setCity("NewCity");
// 3. 하지만 업데이트 쿼리가 2번 나간다.
// 4. 참조를 공유하기 때문에 2번째 회원까지 수정이 된다..
```

#### 객체 타입의 한계
- 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.
- 문제는 임베디드 타입처럼 **직접 정의한 값 타입은 자바의 기본 타입이 아닌 객체 타입이다**.
- 자바 기본타입에 값을 대입하면 값을 복사한다.
- **객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다.**
- **객체의 공유 참조는 피할 수 없다.**

#### 불변객체
- 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단
- 값 타입은 불변 객체로 설계해야함 (immutable Object)
- 불변객체
    - 생성 시점 이후에 절대 값을 변경 할 수 없는 객체
- 불변객체를 만드는 방법
    - 생성자로만 값을 설정하고, 수정자를 만들지 않으면 된다.

> Integer, String 은 자바가 제공하는 대표적인 불변객체

불변이라는 작은 제약으로 부작용이라는 큰 재앙을 막을 수 있다.
