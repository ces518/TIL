# 실전! Querydsl - 동적쿼리 Where 다중 파라미터

#### Where 다중 파라미터
```java
@Test
public void dynamicQuery_WhereParam() {
    String usernameParam = "member1";
    Integer ageParam = 10;

    // 파라미터의 값이 null아냐 아니냐에 따라 쿼리가 동적으로 바뀌어야 한다.
    List<Member> result = searchMember2(usernameParam, ageParam);
    assertThat(result.size()).isEqualTo(1);
}

private List<Member> searchMember2(String usernameParamCond, Integer ageParamCond) {
    return queryFactory
            .select(member)
            .from(member)
            // , 로 구분해서 파라메터를 사용하면 and 로 연결이 된다.
            // null이 들어올경우 조건에서 무시된다.
            .where(usernameEq(usernameParamCond), ageEq(ageParamCond))
            .fetch();
}

private BooleanExpression usernameEq(String usernameParamCond) {
    if (usernameParamCond == null) {
        return null;
    }
    return member.username.eq(usernameParamCond);
}

private BooleanExpression ageEq(Integer ageParamCond) {
    if (ageParamCond == null) {
        return null;
    }
    return member.age.eq(ageParamCond);
}
```
- 실무에서 사용하기 좋다.
- 가독성이 좋아진다.
- 매우 깔끔한 코드가 나온다.

> 가장 강력한 장점은 조건 메소드를 재사용할 수 있고, 조건절을 조립해서 사용이 가능하다.

```java
// 기존 조건 메소드를 재사용해서 조립이 가능하다.
private BooleanExpression allEq(String usernameParamCond, Integer ageParamCond) {
    return usernameEq(usernameParamCond).and(ageEq(ageParamCond));

}
```
