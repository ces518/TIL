# JPA 기본편 - SQL 중심 개발의 문제점

#### 애플리케이션 개발 헤게모니
- 객체지향 언어 [Java, Scala..]

#### 데이터 베이스 세계의 헤게모니
- 관계형 DB [Oracle, MySQL..]

지금 시대는 객체를 관계형 DB에 저장하여 관리해야한다.

#### SQL 중심적인 개발의 문제점
- CRUD
    - 무한 반복적인 노가다..
    - 자바객체를 SQL로, SQL을 자바 객체로 ...

- 객체 CRUD
```java
public class Member {
    private String memberId;
    private String name;
    private String tel;
    ...
}
```

```sql
INSERT INTO MEMBER VALUES(...)
```

필드가 추가되면, 모든 쿼리도 수정이 필요하다.

> **SQL에 의존적인 개발**을 피하기 어렵다.

패러다임의 불일치
- 객체 vs 관계형 데이터베이스

> 객체지향 프로그래밍은 **추상화, 캡슐화, 은닉, 상속, 다형성** 등 시스템의 복잡성을 제어할수 있는 다양한 장치들을 제공한다.

객체를 영구 보관하는 다양한 저장소
- RDB, NoSQL, File ...
- 현실적인 대안은 관계형 DB


객체를 관계형 데이터베이스에 저장
- 객체 -> SQL 변환 -> DB

개발자가 SQLMapper의 일을 하고 있는것..

객체와 관계형 디비의 차이
- 1.상속
- 2.연관관계
    - 객체는 참조관계
- 3.데이터 타입
- 4.데이터 식별 방법


### 연관 관계
- 객체는 참조를 사용한다
    - member.getTeam();
- 테이블은 외래키를 사용한다.
    - JOIN ON M.TEAM_ID = T.TEAM_ID

> 테이블은 Member <-> Team이 가능하지만, 객체는 Member -> Team 만 가능하다.

#### 객체를 테이블에 맞춰 모델링
```java
class Member {
    String id;
    Long teamId;
    String username;
}

class Team {
    Loing id;
    String name;
}
```

#### 객체다운 모델링
```java
class Member {
    String id;
    Team team; // 참조로 연관관계를 맺는다.
    String username;

    Team getTeam () {
        return team;
    }
}

class Team {
    Long id;
    String namel
}
```

#### 객체 모델링, 자바 컬렉션에 관리
```java
list.add(member);

Member member = list.get(memberId);
Team team = member.getTeam();
```

객체로 관리하면 매우 단순하고 깔끔한 문제이지만, DB에 넣고 빼는순간 문제가 발생한다..

#### 객체 그래프 탐색
- 객체는 자유롭게 객체 그래프를 탐색할 수 있어야 한다.
> Delivery Order - Member - Team ...

- 처음 작성한 SQL에 따라 그래프 탐색 이 제한된다.
> 엔티티에 대한 신뢰 문제가 발생한다.
```java
class MemberService {
    public void find () {
        Member member = memberDAO.find(memberId);
        member.getTeam();
        member.getOrder(); // ??
        member.getOrder().getDelivery(); // ??
    }
}
```

Layered 아키텍쳐에서 특정 레이어를 신뢰하고 사용해야하는데 신뢰를 할 수 없게 된다.

> 진정한 의미의 계층 분할이 어렵다. -> 논리적으로 매우 강결합이 되어 있다.


#### 비교하기
```java
String memberId = "100";
Member member1 = list.get(memberId);
Member member2 = list.get(memberId);

member1 == member2;
```

> 객체답계 모델링 할수록 매핑 작업만 늘어난다..

#### 객체를 자바 컬렉션에 저장하듯이 저장할순 없을까 ? 
> JPA (Java Persistence API)
- 1980년대 부터 고민해 왔고 그 고민의 결과가 JPA이다.
