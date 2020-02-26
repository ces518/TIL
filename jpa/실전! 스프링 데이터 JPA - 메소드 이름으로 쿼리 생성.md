# 실전! 스프링 데이터 JPA - 메소드 이름으로 쿼리 생성

#### 메소드 이름으로 쿼리 생성
- Spring data JPA는 메소드 명으로 쿼리를 생성하는 기능을 제공한다.


Username으로 필터링하고, 특정 age보다 큰 조건의 쿼리를 JPA로 구현하면 다음과 같다.
```java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
    return em.createQuery("select m from Member m where m.username = :username and m.age > :age")
            .setParameter("username", username)
            .setParameter("age", age)
            .getResultList();
}
```

위의 쿼리를 Spring data JPA 메소드 쿼리생성 기능을 사용
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsername(String username);

    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

위의 두 메소드 모두 다음과 같은 쿼리가 생성된다.
```sql
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.username=? 
    and member0_.age>?
```

`Spring data JPA가 제공하는 쿼리 메소드 기능`
- `조회`
    - findBy, readBy, getBy..
    - findBy.. 뒤에 조건절이 없다면 findAll() 과 같이 동작한다.
- `count`
    - countBy.. 
    - 반환타입은 long
- `exists`
    - existBy..
    - 반환타입은 boolean
- `삭제`
    - deleteBy, removeBy
    - 반환타입은 long
- `distinct`
    - findDistinct, findMemberDistinctBy
- `limit`
    - findFirst3, findFirst, findTop, findTop3

`쿼리 메소드 필터조건`
- https://docs.spring.io/spring-data/jpa/docs/2.2.4.RELEASE/reference/html/#jpa.query-methods.query-creation

#### 팁
- 짤막한 쿼리들은 메소드명 쿼리생성 기능을 활용한다.
- 실무에서 비중이 크진않음

#### 참고
- 이 기능은 엔티티의 필드명이 변경되면, 인터페이스에 저의한 메소드 명도 꼭 함께 변경해주어야한다.
- 그렇지 않으면 애플리케이션 시작 시점에 오류가 발생하게 된다.
- 애플리케이션 시작 시점에 오류를 인지 할 수 있는것이 Spring data JPA의 매우 큰 장점이다.
