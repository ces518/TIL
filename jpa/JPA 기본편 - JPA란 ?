# JPA 기본편 - JPA 란 ?

#### JPA
- JavaPersistenceAPI
- JAVA 진형의 ORM 표준

#### ORM
- Object-Relational Mapping
- 객체는 객체대로 설계
- 관계형 디비는 관계형 디비대로 설계
- ORM 프레임워크가 중간에서 매핑
- 대중적인 언어에는 대부분 ORM 기술이 존재

> JPA는 애플리케이션과 JDBC 사이에서 동작한다.
- 개발자가 JPA에게 요청을하면 JPA가 JDBC를 활용하여 DB와 통신을 한다.

JPA 저장
- Entity 분석
- INSERT SQL 생성
- JDBC API 생성
- **패러다임 불일치를 해결**

JPA 조회
- SELECT SQL 생성
- JDBC API 사용
- ResultSet 매핑
- **패러다임 불일치를 해결**

#### JPA 소개
- EJB 엔티티빈 (자바 표준) ORM
    - 기술이 너무 아마추어 스러운것이 문제
    - 기능도 좋지 않다.
    - 성능도 좋지 않다.
- 하이버네이트
    - 해외 SI 개발자가 오픈소스로 만들어 낸것
- JPA (자바 표준)
    - 하이버네이트 개발자를 데려와 만들어낸것이 JPA

JPA는 표준 명세
- JPA는 인터페이스의 모음
- JPA2.1 표준명세를 구현한 3가지 구현체
- 하이버네이트, EclipseLink, DataNuclues
    - 대부분이 하이버네이트를 사용

> JPA 와 Spring이 EJB가 불편하여 개발된것이다.

#### JPA를 왜 사용해야하는가
- SQL 중심적인 개발 > 객체중심 개발
- 생산성
- 유지보수
- 패러다임의 불일치 해결
- 성능
- 데이터접근 추상화와 밴더 독립성
- 표준

생산성 - JPA, CRUD
- 저장
    - jpa.persist(member);
- 조회
    - jpa.find(memberId);
- 수정
    - member.setName("asdf");
- 삭제
    - jpa.remove(member);

> JPA 사상 자체가 JAVA Collection에 값을 넣고 빼듯이 사용하는것

유지보수 - JPA, CRUD
- 필드 변경시 기존에는 모든 SQL을 수정해야한다.
- 필드 변경시 Entity만 수정해주면 된다.
    - SQL은 JPA가 처리한다.

JPA 와 패러다임의 불일치 해결
- 상속
    - 상속관계에서 필요한 SQL을 알아서 만들어줌
- 연관관계
    - persist 만 호출해도, 연관관계를 가지고 있는 객체를 저장해준다.
- 객체 그래프 탐색
    - JAVA Collection에 넣었던것 처럼 그래프 탐색 이 가능하다.
- JPA와 비교

신뢰할 수 있는 엔티티, 계층
- 엔티티 계층을 신뢰하고 사용할 수 있다.
```java
public class MemberService {
    public void process () {
        Member memeber = memberDAO.find(memberId);
        member.getTeam();
        member.getOrder().getDelivery();
    }
}
```

비교하기
```java
String memberId = "100";
Membmer a = jpa.find(Member.class, memberId);
Membmer b = jpa.find(Member.class, memberId);
a == b // true
```

> 같은 트랜잭션 내에서는 조회한 엔티티는 같음을 보장 

성능 최적화 기능
- 1차 캐시와 동일성 보장
    - 같은 트랜잭션 내에서 같은 엔티티를 보장 - 조회성능 향상
    - DB Isolation LEVEL 이 Read Commit 이어도 애플리케이션에서 Repetable Read 보장
- 트랜잭션을 지원하는 쓰기 지연
    - 버퍼링 기능
    - 트랜잭션을 커밋할때 까지 INSERT SQL 을 모은다.
    - JDBC BATCH SQL 기능을 사용하여 한번에 SQL 전송한다.
        - SQL시 마다 매번 날리는게 아닌 한번에 모아서 전송한다.
- 지연 로딩과 즉시 로딩
    - 지연로딩:
        - 객체가 실제 사용될때 로딩
    - 지연로딩:
        - JOIN SQL 로 한번에 연관된 객체까지 미리 조회

> ORM은 객체와 RDB 위에 있는 기술이다.
- 객체지향 개발과 RDB 모두 잘 알야아 한다.
