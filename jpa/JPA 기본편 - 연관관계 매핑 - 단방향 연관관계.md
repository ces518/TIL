# JPA 기본편 - 연관관계 매핑 - 단방향 연관관계

### 단방향 연관관계

#### 객체지향 모델링 (객체 연관관계 사용)
- Member
    - id
    - team (team 객체의 참조값)
    - username
- Team
    - id
    - name

- MEMBER
    - MEMBER_ID (PK)
    - TEAM_ID (FK)
    - USERNAME
- TEAM
    - TEAM_ID (PK)
    - NAME

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

//    @Column(name = "TEAM_ID")
//    private Long teamId;
    // 멤버의 입장에서 Many , Team의 입장에서 one 이기 때문에 ManyToOne
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}

@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;
}
```

연관관계 매핑을 @ManyToOne 즉 다대일로 매핑해 준다.
- Member 엔티티 입장에서 Member가 N, Team이 1 이기 때문에 매우 직관적이다.

```sql    
create table Member (
    MEMBER_ID bigint not null,
    USERNAME varchar(255),
    TEAM_ID bigint,
    primary key (MEMBER_ID)
)

create table Team (
    TEAM_ID bigint not null,
    name varchar(255),
    primary key (TEAM_ID)
)

alter table Member 
    add constraint FKl7wsny760hjy6x19kqnduasbm 
    foreign key (TEAM_ID) 
    references Team
```

> 연관관계 매핑을 통한 설계시 생성되는 DDL 또한 FK를 생성해 준다.
