# JPA 기본편 - 프록시와 연관관계 매핑

> 공부를 할때 이걸 왜 써야하지 ? 하고 이유를 알고 써야함

#### 프록시
- Member를 조회할때 Team도 조회해야하나 ?

`회원과 팀을 함께 출력하는 비즈니스 로직`
- 회원과 팀을 함께 출력해야 한다면 회원을 가져올때 한번에 팀까지 가져오면 좋음

`회원만 출력하는 비즈니스 로직`
- 회원만 출력한다면 팀까지 가져오면 성능상 손해임

* 어떤 경우에는 팀과 같이 조회하고, 어떤 경우에는 회원만 가져오고 싶다..

JPA는 이러한 경우는 지연로딩과, 프록시를 사용해서 해결한다.

#### 프록시 기초
- 프록시에 대한 이해가 필요함
- em.find() vs em.getReference()
- em.find()
    - 데이터 베이스를 통해 실제 엔티티 객체를 조회
- em.getReference()
    - 데이터베이스 조회를 미루는 가짜 (프록시) 객체를 조회

```java
Member member = new Member();
member.setUsername("HELLO");

em.persist(member);

em.flush();
em.clear();

Member findMember = em.find(Member.class, member.getId());
System.out.println("findMember.getUsername() = " + findMember.getUsername());
System.out.println("findMember.getId() = " + findMember.getId());

///////////////////////////////////////////////////////////////////////////////
Member findMember = em.getReference(Member.class, member.getId());
System.out.println("findMember.getId() = " + findMember.getId());
System.out.println("findMember.getUsername() = " + findMember.getUsername());
```

> em.find() 는 호출하는 시점에 데이터베이스에서 조회를 하지만, em.getRefernce() 는 값을 사용하는 시점에 데이터베이스에서 조회한다.

#### 프록시 특징
- 실제 클래스를 상속받아 만들어짐
- 실제 클래스와 겉 모양이 같다.
- 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고, 사용하면 된다.
- target 이라는 실제 객체의 참조를 가지고 있다.
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다.

Proxy
- Entity target = null;
- getId()
- getName()

> Proxy객체가 가지고 있는 target이 실제 엔티티이다.


#### 프록시 객체의 초기화
```java
Member member = em.getReference(Member.class, 1L);
member.getName();
```

MemberProxy
- Member target = null;
- getName();

Member 
- name

> - 1.em.getRefernce() 를 호출해서 가짜객체 (Proxy) 에게 값을 요청하면, Proxy객체는 target이 없을경우 영속성 컨텍스트에게 초기화를 요청한다. 
> - 2.영속성 컨텍스트는 데이터베이스에서 조회해서 실제 엔티티객체를 만들어 Proxy객체에게 제공해준다.
- - 3.Proxy객체는 target에 실제값이 존재하게 되고 실제 target객체의 값을 제공해준다.

#### 프록시 객체의 특징
- 프록시 객체는 **처음 사용할때 한 번만 초기화한다.**
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는것은 아니다.
- 초기화 되면 프록시 객체를 통해 실제 엔티티에 접근한다.

- 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야한다. (== 비교시)
    - instance of 를 사용해야한다.

- 영속성 컨텍스트에 찾는 엔티티가 이미 존재한다면, em.getReference() 를 호출해도 실제 엔티티를 반환한다.
    - 이미 원본이 존재하는데 프록시 객체를 만들 이유가 없음
    - 
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일때, 프록시 객체 초기화하면 문제가 발생한다.
- 하이버네이트의 경우에는 (org.hibernate.LazyInitialzationException이 발생한다.)
    - 실무에서 상당히 많이 만나는 예외이다.

> JPA는 한 트랜잭션 내에서 같은 식별자를 가진 엔티티는 동일성 (==) 을 보장한다.

#### 프록시 확인
- 프록시 객체의 상태를 확인하는 유틸 메소드들

- 프록시 인스턴스 초기화 여부확인
    - PersistenceUnitUtil.isLoaded(Object entity);

- 프록시 클래스 확인 방법
    - entity.getClass().getName() 출력
    - ..javasist or HibernateProxy..

- 프록시 강제 초기화
    - org.hibernate.Hibernate.initialize(entity);

- 강제호출
    - member.getName();

> JPA 표준은 강제 초기화가 없다.

em.getReference() 는 실무에서 거의 사용하지 않는다.

하지만 프록시의 지연로딩과 즉시로딩을 어떻게 하는지 이해하기 위해 사전에 프록시에 대한 이해를 돕기 위한 내용
