# 실전! Querydsl - 집합, 그룹핑

#### 집합 함수
```java
@Test
public void aggregation() {
    // 집합 함수를 사용하면 Tuple 타입으로 나온다. * 여러개의 타입이 있을때 꺼내올 수 있음.
    // Querydsl의 Tuple 타입이다.
    // 실무에서는 Tuple보다는 DTO를 많이 사용한다.
    List<Tuple> result = queryFactory
            .select(
                    member.count(), // count
                    member.age.sum(), // sum
                    member.age.max(), // max
                    member.age.min() // min
            )
            .from(member)
            .fetch();

    // Tuple에서 값을 꺼내올때는 select절에서 사용한 표현식을 그대로 사용하면 된다.
    Tuple tuple = result.get(0);
    assertThat(tuple.get(member.count())).isEqualTo(4);
    assertThat(tuple.get(member.age.sum())).isEqualTo(100);
    assertThat(tuple.get(member.age.max())).isEqualTo(40);
    assertThat(tuple.get(member.age.min())).isEqualTo(10);
}
```

- count(), sum(), avg(), max(), min() 등 집합함수 기능을 제공한다.
- select절에서 메소드 체이닝으로 사용가능
- 집합함수를 사용할경우 반환타입은 **Tuple** 이다.
    - Querydsl의 Tuple 타입
    - 여러개의 타입이 존재할때 꺼내서 사용할 수 있다.
- Tupel에서 값을 꺼낼때는 select절에서 사용한 표현식을 그대로 사용하면 된다.

#### 그룹핑
```java
/**
 * 팀의 이름과 각 팀의 평균 연령을 구해라.
 */
@Test
public void group() throws Exception {
    List<Tuple> result = queryFactory
            // select 절에 사용하는것은 QType이어야 한다.
            .select(team.name, member.age.avg()) // 팀의 이름과 팀의 평균연령
            .from(member)
            .join(member.team, team) // member.team과 팀을 조인
            .groupBy(team.name) // 팀 이름으로 그룹핑
//                .having() having 기능도 제공
            .fetch();

    Tuple teamA = result.get(0);
    Tuple teamB = result.get(1);

    assertThat(teamA.get(team.name)).isEqualTo("teamA");
    assertThat(teamA.get(member.age.avg())).isEqualTo(15);

    assertThat(teamB.get(team.name)).isEqualTo("teamB");
    assertThat(teamB.get(member.age.avg())).isEqualTo(35);
}
```

- 집함함수와 동일하게 groupBy, having() 기능을 지원한다.
- 예제에서는 team.name으로 그룹핑했기 때문에 두개의 Tuple이 반환됨.
