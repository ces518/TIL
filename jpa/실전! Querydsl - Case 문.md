# 실전! Querydsl - Case 문

#### Case 문
- 기본적으로 jpql에서 지원하는 case문을 지원한다.
- select, where 절에서 사용 가능

`간단한 조건`
```java
/**
    * 간단한 조건
    */
@Test
public void basicCase() {
    List<String> result = queryFactory
            .select(member.age
                    .when(10).then("10살")
                    .when(20).then("20살")
                    .otherwise("기타"))
            .from(member)
            .fetch();

    for (String s : result) {
        System.out.println("s = " + s);
        /*
            s = 10살
            s = 20살
            s = 기타
            s = 기타
            */
    }
}
```
- 간단한 조건같은 경우 when then otherwise 를 사용해서 간략하게 사용할 수 있음

`복잡한 조건`
```java
/**
    * 복잡한 조건
    */
@Test
public void complexCase() {
    List<String> result = queryFactory
            .select(new CaseBuilder()
                    .when(member.age.between(0, 20)).then("0~20")
                    .when(member.age.between(21, 30)).then("21~30")
                    .otherwise("기타"))
            .from(member)
            .fetch();
    for (String s : result) {
        System.out.println("s = " + s);
        /*
            s = 0~20
            s = 0~20
            s = 21~30
            s = 기타
            */
    }
}
```
- 조건이 복잡해지는 경우 CaseBuilder를 사용해 풀어낼 수 있음


> 이걸 꼭 써야하는가를 먼저 생각할것 
> 가급적이면 DB에서는 이런 문제를 해결하지 않음 
> 최소한의 filtering, grouping, 계산만 할것
