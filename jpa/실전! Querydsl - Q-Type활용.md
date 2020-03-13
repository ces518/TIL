# 실전! Querydsl - Q-Type활용

#### Q-Type 활용
- Q클래스 인스턴스를 활용하는 2가지 방법
```java
@Test
public void startQuerydsl() {
//    QMember m = new QMember("m"); // QMember를 구분하는 이름 (Alias) 크게 중요하진 않음
    QMember m = QMember.member;
    
    // JDBC의 PrepareStatement로 자동으로 파라메터 바인딩을 해준다.
    // > SQL Injection도 방지됨.
    // 컴파일 시점에 오류 체크
    Member findMember = queryFactory
            .select(m)
            .from(m)
            .where(m.username.eq("member1"))
            .fetchOne();

    /*
    static import를 활용한 방법
    Member findMember = queryFactory
            .select(member) 
            .from(member)
            .where(member.username.eq("member1"))
            .fetchOne();
    */


    assertThat(findMember.getUsername()).isEqualTo("member1");
}
```

#### SQL Hint를 활용해 가독성 높히기
```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/querydsl
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
#        show_sql: true
        format_sql: true
        use_sql_comments: true


logging.level:
  org.hibernate.SQL: debug
```

> use_sql_comments= true 옵션을 지정하면 SQL 실행 앞단에 JPQL 주석이 추가된다.

```sql
/* select
    member1 
from
    Member member1 
where
    member1.username = ?1 */ select
        member0_.member_id as member_i1_1_,
        member0_.age as age2_1_,
        member0_.team_id as team_id4_1_,
        member0_.username as username3_1_ 
    from
        member member0_ 
    where
        member0_.username=?
```

> SQL의 Alias 'member1' 은 QMember Generated시 member1이라고 생성되어 member1으로 표출되는것

> 만약 같은 테이블을 조인해서 사용해야 하는경우 new QMember('member2'); 와 같이 새롭게 생성해서 Alias가 중복되지 않게 선언해서 사용할것
