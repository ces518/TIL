# 실전! Querydsl - SQL function 호출하기
- SQL function은 JPA와 같이 Dialect에 등록된 함수만 호출이 가능하다.

#### SQL function 호출하기
```java
@Test
public void sqlFunction() {
    List<String> result = queryFactory
            .select(Expressions.stringTemplate(
                    "function('replace', {0}, {1}, {2})",
                    member.username,
                    "member",
                    "M")) // 회원명에서 member 라는 단어를 M으로 변경하여 조회한다.
            .from(member)
            .fetch();

    for (String s : result) {
        System.out.println("s = " + s);
        /*
        s = M1
        s = M2
        s = M3
        s = M4
            */
    }
    // 임의의 함수를 생성한걸 사용하고 싶다면
    // 기존 Dialect를 상속받는 Dialect를 생성하여 함수를 등록한뒤 해당 Dialect를 사용하면 된다.
}
```

- Expressions.stringTemplate() 메소드를 활용하여 SQL function 호출이 가능하다.
    - 해당 함수가 Dialect에 등록이 되어 있어야 한다. (기본적인 함수들은 모두 등록이 되어있지만 추가적으로 생성한 함수는 별도의 등록이 필요함)

>  만약 커스텀한 함수를 등록해서 사용하고 싶다면 기존 Dialect를 상속받는 Dialect를 생성하여 함수를 등록한 뒤 해당 Dialect를 사용하면 된다.

```java
@Test
public void sqlFunction2() {
    List<String> result = queryFactory
            .select(member.username)
            .from(member)
            .where(member.username.eq(
                    // 모든 DB에서 사용하는 간단한 함수(ANSI 표준함수) 들은 Querydsl에서 내장하고 있다.
//                        Expressions.stringTemplate("function('lower', {0})", member.username)
                    // 다음과 같이 간단하게 사용 가능
                    member.username.lower()
            ))
            .fetch();

    for (String s : result) {
        System.out.println("s = " + s);
    }
}
```
- Expressions.stringTemplate() 를 통한 함수 호출은 매우 불편하다.
- 다행히도 DB에서 사용하는 함수들 (ANSI 표준함수) 들은 Querydsl에서 내장하고 있는것들이 있음.
    - lower() upper() 등..
