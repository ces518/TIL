# 실전! 스프링 데이터 JPA - 순수 JPA 페이징과 정렬

#### 순수 JPA 페이징과 정렬
- 검색조건: 나이가 10살 이상
- 정렬 조건: 이름으로 내림차순
- 페이징 조건: 첫 번째 페이지, 페이지당 보여줄 데이터는 3건

```java
public List<Member> findByPage(int age, int offset, int limit) {
    return em.createQuery("select m from Member m where m.age = :age order by m.username desc", Member.class)
            .setParameter("age", age)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
}

public long totalCount(int age) {
    // totalCount 에서는 sorting 할 필요가 없음
    return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
            .setParameter("age", age)
            .getSingleResult();
}
```

- setFirstResult: 페이징 시작 number
- setMaxResult: 가져올 갯수

> totalCount 에서는 성능을 위해 sorting을 하지않는다. (sorting할 필요 X)
