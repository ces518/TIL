# JPA 기본편 - 서브쿼리

#### 서브쿼리
- JPA에서의 서브쿼리는 일반적인 SQL에서 말하는 서브쿼리와 같은것을 의미한다.

- 나이가 평균보다 많은 회원
```java
select m from Member m
where m.age > (select avg(m2.age) from Member m2)
```

> 메인 쿼리랑 서브쿼리랑 관계가 없는 쿼리, 보통 이렇게 해야 성능이 잘 나온다.

- 한 건이라도 주문한 고객
```java
select m from Member m
where (select count(o) from Order o where m = o.member) > 0
```

> 메인 쿼리랑 서브쿼리가 관계가 있는쿼리, 이런 쿼리는 성능이 잘 안나온다.


#### 서브쿼리 지원 함수
- EXISTS: 서브쿼리에 결과가 존재하면 참이다.
- ALL: 모두 만족시
- ANY, SOME: 같은 의미, 조건중 하나라도 만족시
- IN: 결과중 하나라도 같은 것이 있다면 참

#### JPA 서브 쿼리의 한계
- JPA표준 스펙에서는 where, having 절에서만 서브쿼리가 사용가능하다.
- 구현체인 하이버네이트에서는 select절에서도 사용이 가능하다.
- **from 절의 서브쿼리는 현재 JPQL에서 불가능하다.**
    - 조인으로 해결할 수 있다면 조인으로 해결
    - 대부분의 경우 조인으로 해결이 가능하다.

> from 절에 서브쿼리를 사용하는 경우는 대부분 SQL에 로직이 있는 경우가 많다. 이는 애플리케이션에서 작업함으로써 대부분 해결된다.
