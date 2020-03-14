# 실전! Querydsl - 상수, 문자 더하기

#### 상수, 문자 더하기

`상수`
```java
@Test
public void constant() {
    List<Tuple> result = queryFactory
            .select(member.username, Expressions.constant("A"))
            .from(member)
            .fetch();

    for (Tuple tuple : result) {
        System.out.println("tuple = " + tuple);
        /*
            tuple = [member1, A]
            tuple = [member2, A]
            tuple = [member3, A]
            tuple = [member4, A]
            */
    }
}
```

- Expressions.constant 메소드를 이용해 상수를 사용할 수 있음

`문자 더하기`
```java
@Test
public void concat() {
    // concat()을 사용할때 타입이 맞지 않으면 에러가 발생함.
    // stringValue() 메소드를 이용하여 해결이 가능하다.
    // 문자가 아닌 다른 타입을 처리할때 사용한다.
    // enum Type 같은경우 유용하게 사용된다.
    // member1_10
    List<String> result = queryFactory
            .select(member.username.concat("-").concat(member.age.stringValue()))
            .from(member)
            .fetch();

    for (String s : result) {
        System.out.println("s = " + s);
        /*
            s = member1-10
            s = member2-20
            s = member3-30
            s = member4-40
        */
    }
}
```

- concat() 을 사용해 메소드 체이닝 방식으로 문자열을 더할 수 있음
- 만약 문자열 타입이 아니라면 에러가 발생한다.

`stringValue()`
- Querydsl에서 문자가 아닌 다른 타입을 처리할때 사용한다.
- 특히 enum type을 처리할때 유용하게 사용
