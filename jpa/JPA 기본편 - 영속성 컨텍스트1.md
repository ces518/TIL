# JPA 기본편 - 영속성컨텍스트 1
- JPA를 이해하려면 **영속성 컨텍스트**를 이해해야한다.

#### JPA 에서 가장 중요한 2가지
- 객체와 관계형 데이터베이스 매핑하기 (ORM)
    - 설계와 관련된 부분
- **영속성 컨텍스트**
    - JPA 동작 방식과 관련된 부분

#### 엔티티 매니저 팩토리와 엔티티 매니저
- WebAPP
    - EntityManagerFactory 를 통해 고객의 요청시마다 EntityManager를 생성한다
    - EntityManager를 내부적으로 DB Connection을 사용하여 사용하게 된다.


#### 영속성 컨텍스트
- JPA 이해시 가장 중요한 용어
- **엔티티를 영구 저장하는 환경** 이라는 뜻
- EntityManager.persist(entity);
    - 영속성 컨텍스트를 통해 엔티티를 영속화 한다.
    - DB에 저장하는것이 아니라 영속성 컨텍스트에 저장하는 것이다.


##### 엔티티 매니저 ? 영속성 컨텍스트 ?
- 영속성 컨텍스트는 논리적인 개념
- 눈에 보이지 않는다.
- **엔티티매니저를 통해 영속성 컨텍스트에 접근**한다.

J2SE 환경
- 엔티티 매니자와 영속성 컨텍스트가 1:1로 생성이 된다.

J2EE, Spring 과 같은 컨테이너 환경
- 엔티티 매니저와 영속성 컨텍스트가 N:1 이다.


##### 엔티티의 생명주기
- 비영속 (new/transient)
    - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
    - new 를 통해 새롭게 생성한 객체
- 영속 (managed)
    - 영속성 컨텍스트가 **관리**하는 상태
- 준영속 (detahced)
    - 영속성 컨텍스트에 저장되었다가 분리된 상태
    - 영속성 컨텍스트가 관라하다가 관리대상에서 벗어난 객체
- 삭제 (removed)    
    - 영속성 컨텍스트에서 삭제된 상태


`비영속 상태`
- 영속성 컨텍스트 와 엔티티가 전혀 관련없는 상태
```java
// 객체를 새롭게 생성한 상태 (비영속)
Member member = new Member();
member.setId("Hello");
member.setUsername("안녕?");
```

`영속 상태`
- 영속성 컨텍스트 내에 엔티티가 존재하는 상태
- 영속성 컨텍스트를 통해 엔티티를 관리하는 상태
```java
// 객체를 새롭게 생성한 상태 (비영속)
Member member = new Member();
member.setId("Hello");
member.setUsername("안녕?");

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

// 객체를 저장한 상태 (영속)
// 이 시점에 DB에 저장 되지 않는다.
em.persist(member);
```

> 영속 상태가 된다고 해서 바로 쿼리를 하는 것이 아니다. Transaction을 commit하는 순간 DB에 쿼리를 한다.

`준영속, 삭제`
- 준영속: 영속성 컨텍스트에서 다시 지워버린다.
- 삭제: DB에서 지워버린다.
```java
// 회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
em.detach(member);

// 객체를 삭제한 상태 (삭제)
em.remove(member);
```

#### 영속성 컨텍스트의 이점
- 1차 캐시
- 동일성 보장
- 트랜잭션을 지원하는 쓰기지연
    - write-behind
- 변경감지
    - dirty checking
- 지연로딩
    - lazy loading

> 애플리케이션과 DB 사이에 중간계층이 존재한다. 이를 이용하여 성능적 이점을 가져온다.
