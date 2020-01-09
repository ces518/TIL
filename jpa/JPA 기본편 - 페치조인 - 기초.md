# JPA 기본편 - 페치조인 - 기초

#### 페치 조인 (fetch join)
- SQL 조인의 종류가 아니다.
- JPQL에서 성능 최적화를 위해 제공하는 기능이다.
- 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능이다.
- **join fetch** 명령어를 사용한다.

#### 엔티티 페치 조인
- 회원을 조회하면서 연관된 팀도 함께 조회한다.
- SQL을 보면 회원 뿐 아니라, 팀도 함께 조회한다.

```java
select m from Member m join fetch m.team
```

```sql
select m.*, t.* from member m
inner join team t on m.team_id = t.id
```

> select 프로젝션에 Member 만 명시하였지만, 실제 SQL 에서는 회원과, 팀을 조인해서 같이 가져온다.

- 즉시로딩과 비슷하다.
- 회원과, 팀을 함께 조회되도록 런타임에 동적으로 지정하여 가져오게 된다.


```java
// 이 시점에는 연관관계가 지연로딩 이기때문에 Member와, m.team은 프록시객체를 가지고 있다.
List<Member> members = em.createQuery("select m from Member m", Member.class)
        .getResultList();

for (Member member : members) {
    // getTeam.getName() 을 호출하는 순간, 지연로딩이 되어, 쿼리가 추가적으로 나가게 된다.
    System.out.println("member = " + member.getUsername() + "," + member.getTeam().getName());
    // 회원 100명 조회시 -> 100번 나가게 됨.. (최악의 경우) N + 1 문제가 발생함.
    // 이를 해결하는것이 fetch join
}

// 페치 조인을 해서 team을 한방 쿼리로 가져온다.
// 이때 팀의 정보를 같이 가져 왔기때문에 m.team은 프록시 객체가 아니다.
List<Member> fetchMembers = em.createQuery("select m from Member m join fetch m.team", Member.class)
        .getResultList();

for (Member fetchMember : fetchMembers) {
    // N + 1 문제가 발생하지 않는다.
    System.out.println("fetchMember = " + fetchMember.getUsername() + "," + fetchMember.getTeam().getName());
}
```

#### 컬렉션 페치조인
- 컬렉션조인 1:N 조인을 할경우 SQL에서도 마찬가지로 데이터가 뻥튀기 된다.
    - 카티시안 곱 발생
```java
// 컬렉션 페치 조인
// 일대다 관계, 컬렉션 페치 조인
// * 주의할점 *
// 컬렉션조인 1:N 조인을 할경우 SQL에서도 마찬가지로 데이터가 뻥튀기 된다.
// teamA 소속 회원이 멤버1, 멤버2가 있다면
// teamA 멤버1
// teamA 멤버2
// 위와 같은형태로 로우가 2개로 나오며, teamA가 중복이 된다.
// teamA 엔티티가 중복으로 나온다..
List<Team> teams = em.createQuery("select t from Team t", Team.class).getResultList();

for (Team team : teams) {
    System.out.println("team.getName() = " + team.getName() + "," + team.getMembers().size());
}
```

#### 페치 조인과 DISTINCT
- SQL의 DISTINCT만으로는 중복을 모두 제거할 수 없다.
- JPQL의 DISTINCT
    - 1.SQL에 DISTINCT를 추가해준다.
    - 2.애플리케이션 레벨에서 엔티티의 중복을 제거한다.

```java
// 페치 조인과 DISTINCT
// JPQL의 DISTINCT 2가지 기능 제공
// 1. SQL에 DISTINCT를 추가한다.
// 2. 애플리케이션에서 엔티티 중복을 제거한다.
List<Team> distinctTeams = em.createQuery("select distinct  t from Team t", Team.class)
        .getResultList();
// SQL DISTINCT 만으로는 중복 제거가 되지않아, JPA에서 같은 식별자를 가진 Team 엔티티를 제거해서 원했던 결과가 나온다.
System.out.println("distinctTeams.size() = " + distinctTeams.size());
```
> 같은 식별자를 가진 Team 엔티티를 제거한다.

#### 페치 조인과 일반 조인의 차이
- 일반 조인 실행시, 연관된 엔티티를 함께 조인하지 않는다.
- JPQL은 결과를 반환할때 연관관계를 고려하지 않는다.
- 단지 SELECT 절에 지정한 엔티티에 대한 정보만 조회한다.
```java
// 페치 조인과 일반조인의 차이
// 일반 조인 실행시 연관된 엔티티를 함께 조회하지 않는다.

// 실제 데이터는 t 에 대한 데이터만 조회하게 된다.
// members에는 데이터가 로드되지 않았기 때문에 루프를 돌리면 SQL이 발생하게 된다.
em.createQuery("select t from Team t join t.members m").getResultList();

// 연관된 엔티티를 함께 조회한다.
em.createQuery("select t from Team t join fetch t.members t").getResultList();
```

> 페치조인을 사용할때만, 연관된 엔티티에 대한 정보도 함께 조회한다 (즉시로딩), 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념이다.

* 대부분의 N + 1 문제는 페치조인을 사용해서 해결이 된다.
