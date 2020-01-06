# JPA 기본편 - 페이징

#### 페이징 API
- JPA는 페이징을 다음 두 API로 추상화
- setFirstResult()
    - 조회 시작위치 지정 0부터 시작
- setMaxResult()
    - 조회할 데이터 수

```java
List<Member> members = em.createQuery("select m from Member m order by m.age desc", Member.class)
        .setFirstResult(0)
        .setMaxResults(10)
        .getResultList();

for (Member member : members) {
    System.out.println("member.getUsername() = " + member.getUsername());
}
```

> DB 벤더 별로 페이징 쿼리가 알맞게 나간다.
