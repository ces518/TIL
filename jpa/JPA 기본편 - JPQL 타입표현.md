# JPA 기본편 - JPQL 타입표현

#### JPQL 의 타입표현
- 문자
    - 싱글쿼터로 감싼다.
    - 'HELLO'
- 숫자  
    - 10L (Long)
    - 10D (Double)
    - 10F (Float)
- Boolean
    - true
    - false
- ENUM
    - jpabook.MemberType.Admin
    - 풀패키지 경로가 필요하다.
    - 이는 파라메터 바인딩을 통해 불편함을 해소할 수 있다.
- 엔티티 타입
    - TYPE(m) = Member (상속관계에서 사용)
    - 타입 캐스팅처럼 사용할 수 있음


```java
/**
* JPQL 타입표현
*/
// string, boolean, 숫자는 자바와 동일하고, enum 타입은 풀패키지 경로를 입력해 주어야한다.
List<Object[]> result = em.createQuery("select m.username, 'HELLO', true from Member m where m.type = jpql.MemberType.USER")
    .getResultList();

// 상속관계에서 다음과 같이 엔티티 타입 사용도 가능하다.
em.createQuery("select i from Item i where type(i) = Book");
```

#### JPQL 기타
- SQL 과 문법이 같은 표현식
- EXISTS, IN
- AND OR NOT
- = > >= <= < <>
- BETWEEN, LIKE, IS NULL
- 표준 SQL은 대부분 지원한다고 생각하면 된다.
