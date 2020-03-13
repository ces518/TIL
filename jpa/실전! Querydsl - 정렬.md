# 실전! Querydsl - 정렬

#### 정렬
```java
/**
 * 회원 정렬 순서
 * 1. 나이 내림차순
 * 2. 이름 오림차순
 * 단 2에서 회원 이름이 없으면 마지막에 출력 (nulls last)
 */
@Test
public void sort() {
    em.persist(new Member(null, 100));
    em.persist(new Member("member5", 100));
    em.persist(new Member("member6", 100));

    List<Member> members = queryFactory
            .selectFrom(member)
            .where(member.age.eq(100))
            .orderBy(member.age.desc(),
                    member.username.asc().nullsLast())
            .fetch();
    Member member5 = members.get(0);
    Member member6 = members.get(1);
    Member memberNull = members.get(2);

    assertThat(member5.getUsername()).isEqualTo("member5");
    assertThat(member6.getUsername()).isEqualTo("member6");
    assertThat(memberNull.getUsername()).isNull();
}
```

- 회원 정렬 순서는 다음과 같다.
- 1.나이로 내림차순
- 2.이름으로 오름차순
- 단 2에서 회원이 이름이 null이라면 마지막에 출력 (nulls last)

> Querydsl 에서 where 조건절과 마찬가지로 orderBy도 메소드 체이닝으로 사용이 가능하다.
