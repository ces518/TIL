# 실전! Querydsl - 검색조건 쿼리

#### 검색조건 쿼리
`JPQL이 제공하는 검색조건`
- eq() // ==
- ne() // !=
    - eq().not() 으로 사용가능
- isNotNull()
- in(10, 20)
- notIn(10 ,20)
- betwwen(10, 30)
- goe(30)>=
- gt // >
- loe // <=
- lt // <
- like // member% like 검색
- contains // %member% like 검색
- startWith // member% like 검색

`메소드 체이닝을 활용한 검색조건 작성`
```java
@Test
public void search() {
    Member findMember = queryFactory.
            selectFrom(member)
            .where(member.username.eq("member1")
                    .and(member.age.eq(10)))
            .fetchOne();

    assertThat(findMember.getUsername()).isEqualTo("member1");
    assertThat(findMember.getAge()).isEqualTo(10);
}
```
- username이 'member1'이고, age가 10인 검색조건 쿼리
- where 절에서 메소드 체이닝을 하여 작성이 가능하다.

`메소드 파라메터를 활용한 검색조건 작성`
```java
@Test
public void searchAndParam() {
    Member findMember = queryFactory.
            selectFrom(member)
            .where(
                    member.username.eq("member1"),
                    member.age.eq(10) // and 조건의 경우 파라메터를 ,로 구분하여 넘길경우 and 조건이 적용
                    // null이 들어가게 되면 조건절에서 무시된다.
                    // 동적쿼리 작성시 매우 깔끔하게 적용됨
            )
            .fetchOne();

    assertThat(findMember.getUsername()).isEqualTo("member1");
    assertThat(findMember.getAge()).isEqualTo(10);
}
```

- 메소드 체이닝을 활용한 결과와 동일하게 동작한다.
- ...Predicate 파라메터를 이용해 , 로 구분하여 넘길경우 and 조건이 적용된다.
- null이 들어가게 되면 조건절에서 무시되기 때문에 동적쿼리 작성시 매우 깔끔하게 작성이 가능하다.
