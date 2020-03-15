# 실전! Querydsl - 프로젝션 결과 반환 기본

#### 프로젝션 결과 반환 기본
- 프로젝션: select 절에 대상을 지정하는것

##### 프로젝션 대상이 하나일 경우
```java
/**
    * 프로젝션 대상이 하나일 경우
    */
@Test
public void simpleProjection() {
    List<String> result = queryFactory
            .select(member.username)
//                .select(member) member 엔티티로 조회할 경우에도 프로젝션 대상이 하나이다.
            .from(member)
            .fetch();

    for (String s : result) {
        System.out.println("s = " + s);
    }
}
```
- 프로젝션 대상이 하나면 타입을 명확하게 지정할 수 있다.
- 프로젝션 대상이 둘이상 이라면 튜플이나 DTO로 조회한다.

##### 프로젝션 대상이 둘 이상일 경우
`튜플`
- Querydsl이 프로젝션 대상이 여러 타입 일 때를 대비해서 정의해둔 타입이다.

```java
 /**
    * 프로젝션 대상이 둘이상일 경우 (튜플)
    */
@Test
public void tupleProjection() {
    List<Tuple> result = queryFactory
            .select(member.username, member.age)
            .from(member)
            .fetch();

    for (Tuple tuple : result) {
        String username = tuple.get(member.username);
        Integer age = tuple.get(member.age);
        System.out.println("username, age = " + username + " " + age);
    }
    // Service, Controller 까지 Tuple 타입을 알면 좋지않음
    // 하부 구현기술인 JPA 를 앞단에서 알 수 없도록 하는것이 좋음
    // DTO로 변환해서 내보내는것을 권장
    // Repository 내부에서만 쓰일경우 Tuple 사용
}
```

`참고`
- Tuple은 Querydsl에 종속적인 타입이다.
- Repository 계층 내부에서만 쓰일때는 문제가 없지만, Service, Controller로 그대로 반환할 경우에는 문제가 된다.
- 하부 구현기술인 JPA가 Service, Controller에 노출되기 때문에 좋지 않은 설계이다.
- 구현 기술이 바뀌면 서비스, 컨트롤러 모두 변경되어야한다.
    - 하부 구현기술이 앞단에 노출되지 않는것이 좋은 설계
- DTO로 변환해서 내보내는것을 권장한다.
