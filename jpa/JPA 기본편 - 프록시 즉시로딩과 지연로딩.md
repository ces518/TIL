# JPA 기본편 - 프록시 즉시로딩과 지연로딩

#### Member를 조회할때 Team도 함께 조회해야 할까 ?
- 단순히 member만 조회해야하는 비즈니스 로직이라면 ?
    - 연관관계가 걸려있다고 해서 항상 같이 조회한다면 손해이다.

#### 지연로딩 LAZY를 사용해서 프록시로 조회
```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    private Team team;
}
```
- FetchType을 LAZY로 설정하게되면 , 해당 엔티티 는 실제 엔티티가 아닌 프록시 객체를 사용한다.
    - member.getTeam().getClass() 를 하면 프록시 객체가 나온다.

#### 지연 로딩
- 기본 매커니즘
    - Member를 로딩할때 Member는 실제 엔티티를, Team은 프록시 객체를 가져온다.
    - Team을 사용하는 시점에 영속성 컨텍스트를 통해 Team을 조회한다.

> 비즈니스 로직 대부분이 Member만 사용하는게 대부분이라면 지연로딩 형태로 가는것이 맞다.

#### 즉시로딩 EAGER를 사용해서 함께 조회
- Member 와 Team이 항상 함께 사용된다면 ?
```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @ManyToOne(fetch = FetchType.EAGER)
    private Team team;
}
```

> Member를 조회할때 Team까지 함께 조회하기 때문에 프록시 객체가 필요 없다. 따라서 Team은 프록시 객체가 아닌 실제 엔티티 객체가 된다.

> JPA 구현체는 가능하면 조인을 사용해서 SQL 한번에 함께 조회한다.

#### 프록시와 즉시로딩 주의
- 가급적 지연 로딩만 사용 (특히 실무..)
- 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생한다.
- 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
- @ManyToOne, @OneToOne 은 기본이 즉시 로딩 이다.
    - LAZY로 변경할것
- @OneToMany, @ManyToMany 는 기본이 지연 로딩

- em.find() 는 PK를 넣어서 단건을 조회하기 떄문에 내부적으로 최적화가 가능하다.
- 하지만 JPQL에서는 SQL로 일단 변환을 하고 쿼리를 한다.
    - 이때 연관관계가 EAGER라면 해당 엔티티에 대한 쿼리를 N 번 만큼 더 발생하게 됨

`해결방법`
1. Fetch Join
    - 매 상황마다 동적으로 로딩함
        - Member 만 조회한다면 Member만, Team을 함께 사용한다면 Team과 함께 조인해서 가져온다
    - 이 방법으로 대부분 해결을 한다.
2. @EntityGraph
3. Batch Size


> 연관관계 매핑은 모두 LAZY를 사용할것 !! , 실무에선 JPQL을 많이 사용하는데 JPQL에서 문제가 발생함

#### 지연로딩 활용
- Member, Team 함께 자주사용 -> 즉시 로딩
- Member, Order 는 가끔 사용 -> 지연로딩
- Order, Product 는 자주 함께사용 -> 즉시로딩

> 위의 내용은 이론적인 내용이고, 실무에선 모두 지연로딩을 사용할것

#### 지연로딩 활용 - 실무
- 모든 연관관계에서 지연로딩을 사용할것
- 실무에서는 즉시로딩을 사용하지 말것
- JPQL fetch 조인, 엔티티 그래프 기능을 사용
- 즉시 로딩은 상상하지 못한 쿼리가 나간다.
