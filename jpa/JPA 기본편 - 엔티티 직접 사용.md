# JPA 기본편 - 엔티티 직접 사용

#### JPQL 엔티티 직접 사용 - 기본 키 값
- JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키 값을 사용한다.
```java
select count(m.id) from Member m// 엔티티의 아이디를 사용
select count(m) from Member m // 엔티티를 직접 사용한다.
```
```sql
select count(m.id) as cnt from Member m
```

`엔티티를 파라미터로 전달`
```java
select m from Member m where m =:member;
```

`식별자를 직접 전달`
```java
select m from Member m where m =:id;
```

> 위의 JPQL 모두 실행되는 SQL은 동일하다.

#### JPQL 엔티티 직접 사용 - 외래 키 값
```java
// 외래 키값 사용
em.createQuery("select m from Member m where m.team = :team", Member.class)
    .setParameter("team", teamA)
    .getResultList();
```

> JPQL에서 기본키값으로 엔티티를 직접사용하는것 처럼 식별자가 SQL에서 사용된다.
