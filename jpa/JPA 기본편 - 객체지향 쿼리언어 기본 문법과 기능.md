# JPA 기본편 - 객체지향 쿼리언어 기본 문법과 기능

#### JPQL - 소개
- JPQL은 객체지향 쿼리 언어이다.
- 테이블을 대상으로 쿼리하는 것이 아닌 **엔티티 객체를 대상으로 쿼리**한다.
- JPQL은 SQL을 추상화해서 **특정 데이터베이스 SQL에 의존하지 않는다**.
- JPQL은 결국 SQL이다.

[객체모델]
Member
    - id
    - username
    - age

Team
    - id
    - name

Order
    - id
    - orderAmount
    - address

Product
    - id
    - name
    - price
    - stockAmount

`Value Type`
Address
    - city
    - street
    - zipcode


[DB모델]
MEMBER
    - ID (PK)
    - TEAM_ID (FK)
    - NAME
    - AGE

TEAM
    - ID (PK)
    - NAME

ORDERS
    - ID (PK)
    - ORDER_ID (FK)
    - PRODUCT_ID (FK)
    - ORDERAMOUNT
    - CITY
    - STREET
    - ZIPCODE

PRODUCT
    - ID (PK)
    - NAME
    - PRICE
    - STOCKAMOUNT

#### JPQL 문법
- JPQL 문법은 SQL과 동일하다.
    - select, from .. update.. 등
- 벌크연산
    - 전 사원의 연봉을 10% 인상 시킨다 등.
    - 한번에 여러개를 업데이트 시킬때

```java
select m from Member as m where m.age > 18
```
- 엔티티와 속성은 대소문자를 구분한다.
- JPQL 키워드는 대소문자를 구분하지 않는다.
- 엔티티 명을 사용해야한다, 테이블 명이 아니다 
- 별칭은 필수이다. (as 는 생략 가능)

#### 집합과 정렬
```java
select
    count(m),
    sum(m.age),
    avg(m.age),
    max(m.age),
    min(m.age)
from Member m
```
- 모든 집합과 정렬 함수는 SQL과 동일하게 사용할 수 있다.

#### TypeQuery, Query
- TypeQuery: 반환 타입이 명확할때 사용한다.
- Query: 반환 타입이 명확하지 않을때 사용한다.

```java
TypedQuery<Member> memberQuery = em.createQuery("select m from Member m", Member.class);
// username은 String, age 는 int 이기 때문에 타입 정보를 명확하게 줄 수 없다.
Query query = em.createQuery("select m.username, m.age from Member m");
```

#### 결과 조회 API
```java
// 결과값을 반환시 한개 이상의 결과를 받고싶다면  getResultList() 를 사용한다.
List<Member> members = memberQuery.getResultList();

// 결과값을 반드시 하나로 받고 싶을때 getSingleResult() 를 사용한다.
Member singleMember = memberQuery.getSingleResult();
```
- query.getResultList();
    - 결과 값이 하나 이상일때 사용한다.
    - 결과 값이 없다면 빈 리스트를 반환한다.
- query.getSingleResult();
    - 결과가 정확히 하나일떄 사용한다.
    - 단일 객체를 반환한다.
    - 결과가 없다면 javax.persistence.NoResultException 발생
    - 둘 이상이라면 javax.persistence.NonUniqueResultException 발생

> getSingleResult는 반드시 결과값이 하나라고 확신될때 사용할것

* Spring Data JPA 에서는 단일건 함수들을 추상화 해서 제공한다. 결과가 없을시 Optional, null 을 반환하고, 예외를 발생하지 않는다.
- 표준 스펙이기 때문에 Spring Data JPA 에서도 이를 사용해야한다. 코드를 까보면 try-catch 로 null or Optional을 반환해주는 식으로 구현이 되어있다.

#### 파라미터 바인딩 - 이름 기준, 위치 기준
```java
// 파라미터 바인딩
TypedQuery<Member> memberTypedQuery = em.createQuery("select m from Member m where m.username = :username", Member.class);
// setParameter 를 활용하여 바인딩 해준다.
memberTypedQuery.setParameter("username", "member1");
Member singleResultMember = memberTypedQuery.getSingleResult();
```

> 위치기반 바인딩도 제공을 하지만, 이는 사용하지 않는것을 권장한다. 중간에 파라메터가 껴버리면 순서가 변경되어 버그 발생 여지가 있다.
