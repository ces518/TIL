# 실전! Querydsl - 프로젝션 결과 반환 @QueryProjection

#### 생성자 + @QueryProjection
- compileQuerydsl task를 실행해서 DTO를 QType으로 생성해야한다.

`QMemberDto`
```java
@Generated("com.querydsl.codegen.ProjectionSerializer")
public class QMemberDto extends ConstructorExpression<MemberDto> {

    private static final long serialVersionUID = 1356709634L;

    public QMemberDto(com.querydsl.core.types.Expression<String> username, com.querydsl.core.types.Expression<Integer> age) {
        super(MemberDto.class, new Class<?>[]{String.class, int.class}, username, age);
    }

}
```

```java
@Test
public void findDtoByQueryProjection() {
    // 이전의 생성자 사용 방식의 단점은 런타임시 에러가 발생한다.
    // @QueryProjection 방식은 생성자를 그대로 가져오기 때문에 타입을 안정적으로 바인딩 가능
    // 컴파일 시점에 오류도 잡아줌
    List<MemberDto> result = queryFactory
            .select(new QMemberDto(member.username, member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }

    // 단점
    // QType을 생성해야함.
    // MemberDto가 Querydsl에 의존적이게 된다.
    // 보통 DTO는 여러 레이어에 걸쳐서 사용이 되는데.
    // 순수한 DTO가 아니게 된다. (아키텍쳐 적인 문제)
}
```
- **기존의 생성자 사용 바인딩 방식의 단점은 런타임시에 에러 체크가 가능하다.**
- @QueryProjection 방식은 생성자를 그대로 사용하기 때문에 IDE의 도움을 받을수 있어 안정적인 바인딩이 가능
    - 컴파일시점에 오류 체크가 가능하다.

#### 단점
- QType을 생성해야한다.
- 아키텍쳐적인 문제가 발생함.
- MemberDto는 Querydsl에 의존적이게 된다.
- 보통 DTO는 여러 레이어에 걸쳐서 사용하게 되는데 순수한 DTO가 아니게 된다.
    - Querydsl 제거시 DTO가 영향을 받게됨.

> 아키텍쳐 적인 문제를 트레이드 오프하면 매우 안정적이고, 가장 편리한 방법이다.
