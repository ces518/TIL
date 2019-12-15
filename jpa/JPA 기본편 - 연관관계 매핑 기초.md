# JPA 기본편 - 연관관계 매핑 기초

### 연관관계 매핑
- 객체와 RDB의 패러다임의 차이 간에 오는 어려움이 오는 구간
- 객체와 테이블 연관관계 차이를 이해
- 객체의 참조와 테이블의 외래키를 매핑

- 방향: 단방향, 양방향
- 다중성: 다대일, 일대다, 일대일, 다대다
- 연관관계의 주인: 객체 양방향 연관관계는 관리하는 주인이 필요

#### 연관관계가 필요한 이유
- 객체지향 설계의 목표는 자율적인 객체들의 **협력 공동체**를 만드는 것이다.

#### 예제 시나리오
- 회원과 팀이 있다.
- 회원은 하나의 팀에만 소속될 수 있다.
- 회원과 팀은 다대일 관계이다.

#### 객체를 테이블에 맞추어 모델링 (연관관계가 없는 객체)
- Member
    - id
    - teamId
    - username
- Team
    - id
    - name

- MEMBER
    - MEMBER_ID (PK)
    - TEAM_ID (FK)
    - USERNAME
TEAM
    - TEAM_ID (PK)
    - NAME


> 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.
- 테이블은 외래키 조인을 사용하여 연관관계를 맺는다.
- 객체는 참조를 사용해 연관관계를 맺는다.
- 테이블과 객체 사이에는 이러한 큰 차이점이 존재한다.

`데이터중심 모델링 결과 `
```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

   @Column(name = "TEAM_ID")
   private Long teamId;
}
```

다음과 같은 DDL 이 생성된다.
```sql   
create table Member (
    MEMBER_ID bigint not null,
    TEAM_ID bigint,
    USERNAME varchar(255),
    primary key (MEMBER_ID)
)

create table Team (
    TEAM_ID bigint not null,
    name varchar(255),
    primary key (TEAM_ID)
)
```

> Member 와 Team 간의 관계가 맺어지지 않음.. 잘못된 설계
