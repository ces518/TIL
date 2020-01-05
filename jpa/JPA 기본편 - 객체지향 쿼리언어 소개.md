# JPA 기본편 - 객체지향 쿼리언어 소개

#### JPA는 다양한 쿼리 방법을 지원
- 복잡한 쿼리를 사용할 수 있게끔 지원한다.
- **JPQL**
    - 표준 문법
- JPA Criteria
    - JPQL을 자바코드로 만들수 있게 한다
- **QueryDSL**
    - JPQL을 자바코드로 만들수 있게 한다
- 네이티브 SQL
    - 데이터베이스에 종속적인 쿼리를 사용해야할때 사용한다.
    - 표준 SQL을 벗어나는 문법이 필요할때 사용한다.
- JDBC API 직접사용, Mybatis, SpringJdbcTemplate 함께 사용
    - JPA 를 사용하면서 같이 사용할 수 있다.

> 대부분의 문제는 JPQL로 해결이 되지만, 네이티브 쿼리나 Mybatis 등을 사용해서 해결해야할 경우가 간혹 등장한다.

#### JPQL 소개
- 가장 단순한 조회 방법
    - em.find()
    - a.getB().getC() ..

- 나이가 18살 이상인 회원을 모두 검색하고 싶을때 ?..

#### JPQL
- JPA를 사용하면 엔티티 객체 중심으로 개발
- 문제는 검색쿼리이다..
- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야 한다.
- 모든 DB데이터를 객체로 변환해서 검색하는것은 불가능하다.
- 애플리케이션이 필요한 데이터만 조회해오려면 결국 검색조건이 포함된 SQL이 필요하다.

##### 객체지향 쿼리 ?
- JPA는 SQL을 추상화한 JPQL이라는 객체지향 쿼리 언어를 제공한다.
- SQL 문법과 유사하며 ANSI 표준이 지원하는 문법을 모두 지원한다.
- JPQL은 **엔티티 객체를 대상으로 쿼리**한다.

```java
List<Member> members = em.createQuery("select m from Member m where m.username like '%kim%'",
        Member.class)
        .getResultList();
```

- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리이다.
- SQL을 추상해서 특정 데이터베이스 SQL에 의존하지 않는다.
- 한마디로 정의한다면 객체지향 SQL

##### JPQL의 문제점
- JPQL은 결국 문자열이다.
- 휴먼에러 (개발자의 실수에 의한 에러) 가 발생할 수 있다.
- 동적쿼리를 생성하기 힘들다.

#### Criteria 소개
- 문자가 아닌 자바코드로 JPQL을 작성할 수 있다.
- JPQL 빌더 역할
- JPA 공식 기능이다.
- 자바 표준에서 제공하는 기능이다.
- 동적쿼리를 생성할때 장점이 있다.
- 컴파일시점에 IDE에서 잡아주기 때문에 이점이 있다.

```java
/** Criteria */
// 자바 표준에서 제공하는 기능이다.
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

// 루트클래스 (조회를 시작할 클래스)
Root<Member> m = query.from(Member.class);

// 동적쿼리 생성
CriteriaQuery<Member> cq = query.select(m)
        .where(cb.equal(m.get("username"), "kim"));

List<Member> members = em.createQuery(cq).getResultList();
// 쉬운것 같으면서도 어렵다..
```

> 실무에서 사용하기 어렵다. 유지보수가 힘들다.. QueryDSL을 사용할 것을 권장한다.


#### QueryDSL
- 컴파일 시점에 IDE에서 잡아주기 때문에 이점이 있다.
- 문자가 아닌 자바코드로 JPQL로 작성할 수 있다.
- JPQL 빌더 역할
- 동적쿼리 작성이 편하다
- 단순하고 쉽다
- 실무 에서 사용을 권장한다.

```java
QMember m = QMember.member;

queryFactory
    .select(m)
    .from(m)
    .where(m.name.like("kim"))
    .fetch();
```

> JPQL을 잘하면 QueryDSL은 공짜로 가져가는 셈이다.

#### 네이티브 SQL 소개
- JPA가 제공하는 SQL을 직접 사용하는 기능이다
- JPQL로 해결할 수 없는 특정 데이터비에스 에 의존적인 기능이다.

```java
em.createNativeQuery("select MEMBER_ID, city, street, zipcode from MEMBER").getResultList();
```

#### JDBC 직접사용, SpringJdbcTemplate 등..
- JPA를 사용하면서 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스 등을 함께 사용 가능하다.
- 단, 영속성 컨텍스트를 직절한 시점에 강제로 플러시가 필요하다.
- JPA를 우회해서 SQL을 실행하기 직전에 컨텍스트 수동 플러시가 필요

- JPA와 관련된 기술을 사용할땐 문제가 되지 않는다.
    - 영속성 컨텍스트의 flush는 커밋 또는 쿼리가 나가는 시점에 자동 flush 된다.
    - createQuery, createNativeQuery 등 ..
    
> JdbcTemplate, 마이바티스 등은 JPA와 관련이 없는 기술이기 때문에 JPA의 영속성 컨텍스트의 라이프사이클과 별개로 동작한다. 따라서 강제로 플러시를 해주는 작업이 필요함.
