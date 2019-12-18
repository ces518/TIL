# JPA 기본편 - 연관관계 매핑 - 양방향 연관관계와 연관관계의 주인 - 주의점

#### 양방향 매핑시 가장 많이하는 실수
- 연관관계의 주인에 값을 입력하지 않음
    - 연관관계가 주인이 아니면 DB값에 영향을 주지않음.
```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setName("memberA");

// team에만 멤버를 추가해 주었음.
team.getMembers.add(member);
em.persist(member);
```

#### 양방향 매핑시 연관관계의 주인에 값을 입력해야 한다.
- 순수한 객체관계를 고려한다면 양쪽 다 값을 입력해야 한다.
```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setName("memberA");

// 객체지향적으로 생각을 해본다면 양쪽에 다 주는것이 맞다.
member.setTeam(team);
team.getMembers.add(member);

em.persist(member);
```

- 양쪽 다 넣어 주지 않는다면 두 군데에서 문제가 발생한다.
    - Member에는 team을 지정해주고, Team의 members에는 추가해 주지 않는 상황일때
    - 영속성 컨텍스트를 flush, clear 해주지 않는다면 1차캐시에 남아 있게 된다.
    - 1차캐시에서 조회한 값을 그대로 조회해서 사용하기 때문에 양쪽 다 넣어주지 않는다면 team의 members에는 해당 멤버가 존재하지 않은 상태로 나오게 된다.
- 또 하나는 테스트 케이스를 작성할때 문제가 발생한다.

> 객체지향적으로 생각한다면 양쪽다 값을 세팅 해주는 것이 맞다.



#### 주의점
- 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정
- 연관관계 편의 메소드를 생성
- 양방향 매핑시 무한 루프를 조심
    - toString, lombok, JSON 생성 라이브러리

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

    /**
     * 연관관계 편의 메소드
     * @param team
     */
    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```

> 연관관계 편의 메소드도 양쪽에 다 있으면 문제를 일으킬 수 있기때문에 편의 메소드는 한쪽에만 생성하도록 한다.

#### TIP
- Entity를 그대로 반환하지 말것
- Entity는 DTO로 변환해서 반환하는것을 추천한다.
    - Entity가 변경되면 API스펙이 변경되어 버린다.

#### 양방향 매핑 정리
- **단뱡항 매핑으로도 이미 연관관계 매핑은 완료**
- 양방향 매핑은 반대방향으로 조회(객체 그래프 탐색) 기능이 추가된것 일뿐
- 단방향 매핑으로 설계를 끝내야 한다.
- JPQL에서 역방향으로 탐색할 일이 많다.
- 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 됨
    - 테이블에 영향을 주지 않는다.

#### 연관관계의 주인을 정하는 기준
- 비즈니스 로직을 기준으로 연관관계의 주인을 선택하면 안됨
- 연관관계의 주인은 외래 키의 위치를 기준으로 정해야함.
