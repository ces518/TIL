# JPA 기본편 - 영속성컨텍스트 2

#### 엔티티조회 ,1차캐시
- 영속성 컨텍스트는 내부적으로 캐싱을 한다.
- 1차캐시를 영속성 컨텍스트라고 이해해도 된다.
```java
// 엔티티를 생성한 상태 (비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원");

// 1차캐시에 저장됨
em.persist(member);

// 1차캐시에서 조회
Member findMember = em.find(Member.class, "member1");
```

> 먼저 DB에서 조회하는것이 아닌 1차캐시(영속성 컨텍스트) 에서 먼저 찾고, 없다면 DB에서 조회한다. DB에서 조회한 데이터는 영속성 컨텍스트에 1차 캐시를 한다.

보통 DB Transaction 단위로 처리를 하기때문에 **1차캐시**는 엄청나게 큰 도움이 되진 않는다.
- 비즈니스 로직이 복잡한 경우에는 도움이 된다.

#### 영속 엔티티의 동일성 보장
```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");
a == b // true
```

JPA는 영속 엔티티의 동일성을 보장해준다.
- 1차 캐시가 존재하기 떄문에 가능한 것이다.
- 1차 캐시로 반복 가능한 읽기(Repeatable READ) 등급의 트랜잭션 격리 수준을 DB가 아닌 애플리케이션 레벨에서 제공해준다.

#### 엔티티 등록 - 트랜잭션을 지원하는 쓰기 지연
```java
EntityManager em - emf.createEntityManager();
EntityTrasaction tx = em.getTransaction();
tx.begin();

em.persist(memberA);
em.persist(memberB);
// 여기까지 등록 쿼리를 보내지 않는다.

tx.commit();
// 트랜잭션을 커밋하는 순간에 쿼리를 한다.
```

- 영속성 컨텍스트 내에는 쓰기지연 SQL 저장소 라는 공간이 존재한다.
- persist 할때 마다 SQL 저장소에 쌓아 둔다. (이 때 DB에는 질의하지 않는다.)
- 트랜잭션이 commit 될때 쌓아둔 SQL을 DB에 질의한다.
    - JDBC Batch
    - hibernate.jdbc.batch_size 옵션이 존재함
- 한번의 통신으로 모아둔 쿼리를 날린다.


> TIP JPA 는 내부적으로 Reflection등 의 기술을 사용하기 때문에 엔티티 클래스에는 기본 생성자가 하나 존재 해야한다.

#### 엔티티 수정 - 변경 감지
```java
EntityManager em = emf.createEntityManager();
EntityTransaction tx em.getTransaction();
tx.begin();

Member findMember = em.find(Member.class, 10L);
findMember.setName("HELLO");
// 변경 감지로 인해 업데이트 쿼리가 자동적으로 나간다.

tx.commit();
// 내부적으로 flush가 호출된다.
```

> 커밋 하는 순간에 엔티티와 스냅샷을 비교한다.
- 스냅샷: JPA는 값을 최초로 읽어온 순간을 스냅샷을 떠둔다.


#### 엔티티 삭제
```java
Member memberA = em.find(Member.class, 100L);
em.remove(memberA);
```
