# JPA 기본편 - 다양한 연관관계 매핑 - 일대다

#### 일대다 단방향
- 팀을 중심으로 매핑
- Team  
    - id
    - name
    - List members
- Membmer
    - id
    - username
    - Team team
- TEAM
    - TEAM_ID (PK)
    - NAME
- MEMBER    
    - MEMBER_ID (PK)
    - TEAM_ID (FK)
    - USERNAME

> 이 모델은 권장하지 않는다.
- 표준 스팩에서 지원을 하지만 실무에서는 되도록 사용하지 않는것을 추천함

- Team의 members가 변경되었을때 다른 테이블에있는 MEMBER테이블의 외래키를 update 쳐야한다. 
    - 이상 ?..

> 이런 형태보다는 Member를 연관관계의 주인으로 지정해서 (Trade Off) 사용하는것이 좋음

```java
@Entity
public class Team {
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();
}
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;
}
```

#### 일대다 단방향 정리
- 일대다 단방향은 일이 연관관계의 주인이다
- 테이블의 일대다 관계는 항상 다 쪽에 외래 키가 있다
- 객체와 테이블의 차이 떄문에 반대편 테이블의 외래 키를 관리해야한느 특이한 구조
- @JoinColumn을 꼭 사용해야한다.
    - 그렇지 않으면 조인 테이블방식을 사용한다.

- 단점
    - 엔티티가 관리하는 외래키가 다른 테이블에 있음
    - 연관관계 관리를 위해 추가로 SQL이 실행됨

> 객체적으로 손해를 보더라도, 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자


#### 일대다 양방향
- 팀을 중심으로 매핑 - 약간 억지성으로 된다.
    - 스펙상 지원되는것은 아니다.

- Team  
    - id
    - name
    - List members
- Membmer
    - id
    - username
    - Team team (읽기전용 매핑)
- TEAM
    - TEAM_ID (PK)
    - NAME
- MEMBER    
    - MEMBER_ID (PK)
    - TEAM_ID (FK)
    - USERNAME

#### 일대다 양방향 정리
- 이런 매핑은 공식적으로 존재하지 않음
- @JoinColum(insertable = fasle, updatable = false)
- 읽기 전용 필드를 사용해 양방향처럼 사용하는 방법
- 다대일 양방향을 사용하자

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;
}
```
