# 실전! Querydsl - 프로젝션 결과 반환 DTO

#### 프로젝션 결과 반환 DTO

##### 순수 JPA
```java
@Test
public void findDtoByJPQL() {
    // new Operation 을 활용하는 방법
    // DTO의 생성자가 호출된다.
    List<MemberDto> result = em.createQuery("select new study.querydsl.dto.MemberDto(m.username, m.age) " +
                    "from Member m",
            MemberDto.class)
            .getResultList();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```
- 순수 JPA에서 DTO로 조회할 때는 new 명령어를 사용해야 함
- DTO의 패키지 명을 적어줘야 해서 지저분함
- 생성자 방식만 지원한다.


##### Querydsl
3가지 방식 모두 지원
- 프로퍼티 접근
- 필드 직접 접근
- 생성자 사용

`프로퍼티 접근 방식`
```java
@Test
public void findDtoBySetter() {
    // 프로퍼티 접근 방식
    // setter를 활용해서 데이터를 바인딩한다.
    List<MemberDto> result = queryFactory
            .select(Projections.bean(MemberDto.class,
                    member.username,
                    member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```
- Projections.bean 메소드를 사용해서 값을 바인딩 한다.
- 프로젝션에 명시된 값과 DTO의 프로퍼티명이 일치해야 한다.

`필드 직접 접근`
```java
@Test
public void findDtoByField() {
    // 필드 접근 방식
    // 필드에 다이렉트로 값을 바인딩 해준다.
    List<MemberDto> result = queryFactory
            .select(Projections.fields(MemberDto.class,
                    member.username,
                    member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```
- Projections.fields 메소드를 사용해서 값을 바인딩 한다.
- 프로젝션에 명시된 값과 DTO의 필드명이 일치해야 한다.

`생성자 사용`
```java
@Test
public void findDtoByConstructor() {
    // 생성자 사용 방식
    // 생성자의 파라미터 순서를 맞춰주어야 함
    List<MemberDto> result = queryFactory
            .select(Projections.constructor(MemberDto.class,
                    member.username,
                    member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```
- Projections.constructor 메소드를 사용해 값을 바인딩 한다.
- 프로젝션에 명시된 값과 DTO의 생성자 파라미터 순서를 일치 시켜주어야 한다.

`필드(프로퍼티) 명이 다를 경우`
```java
@Test
public void findUserDto() {
    // 필드 접근 방식
    // 필드에 다이렉트로 값을 바인딩 해준다.
    // DTO의 속성과 프로젝션에 정의된 명이 일치해야 값이 바인딩 된다.
    // 만약 일치하지 않을경우 .as() 메소드를 이용해 별칭을 지정해 주어야함

    QMember memberSub = new QMember("memberSub");

    List<UserDto> result = queryFactory
            .select(Projections.fields(UserDto.class,
                    member.username.as("name"),

                    // 필드 혹은 서브쿼리의 별칭을 지정해 줄때 ExpressionUtils를 사용하여 줄 수 있음
                    ExpressionUtils.as(JPAExpressions
                            .select(memberSub.age.max())
                            .from(memberSub), "age"))
            )
            .from(member)
            .fetch();

    for (UserDto userDto: result) {
        System.out.println("userDto = " + userDto);
    }
}
```
- **필드 접근 방식(프로퍼티 접근 방식도 동일)**에서는 일치하지 않을경우 바인딩이 되지 않는다.
    - as() 메소드를 이용해 별칭을 지정해 주어야한다.
- 서브쿼리를 사용할때도 동일한 문제가 발생한다.
    - ExpressionUtils.as() 메소드를 이용해 별칭을 지정해 줄 수 있다.
    - 필드에 별칭을 지정해줄때도 사용할 수 있다.
- 생성자 사용방식은 프로퍼티 명이 일치하지 않고, 순서만 일치한다면 바인딩 된다.
