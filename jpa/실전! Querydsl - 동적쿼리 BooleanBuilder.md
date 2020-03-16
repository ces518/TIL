# 실전! Querydsl - 동적쿼리 BooleanBuilder

#### 동적쿼리를 해결하는 두가지 방식
- BooleanBuilder
- Where 다중 파라미터 사용

> 일반적인 Querydsl 문서를 보면 BooleanBuilder를 사용하는 방법이 소개된다.

#### BooleanBuilder
```java
@Test
public void dynamicQuery_BooleanBuilder() {
    String usernameParam = "member1";
    Integer ageParam = 10;

    // 파라미터의 값이 null아냐 아니냐에 따라 쿼리가 동적으로 바뀌어야 한다.
    List<Member> result = searchMember1(usernameParam, ageParam);
    assertThat(result.size()).isEqualTo(1);
}


private List<Member> searchMember1(String usernameParamCond, Integer ageParamCond) {

    // 초기값을 넣어즐 수있음.
    BooleanBuilder builder = new BooleanBuilder(member.username.eq(usernameParamCond));

    if (usernameParamCond != null) {
        builder.and(member.username.eq(usernameParamCond));
    }
    if (ageParamCond != null) {
        builder.and(member.age.eq(ageParamCond));
    }

    return queryFactory
            .select(member)
            .from(member)
            .where(builder)
            .fetch();
}
```
- 동적쿼리를 생성하는 빌더역할을 한다.
- BooleanBuilder 객체를 생성할때, 생성자 파라메터로 초기값을 지정해 줄 수 있다.
- and, or 등 다양한 연계연산이 가능하다.


> Mybatis(ibatis)를 사용하는 방법과 유사해서 깔끔해 보이지 않을 수 있다. BooleanBuilder를 사용해야 할 때와, Where 다중 파라메터를 사용할때가 다름
