# 실전! Querydsl - 소개

#### 소개
- 최신 자바 기술은 Spring Boot + Spring data JPA 
> 복잡한 쿼리, 동적 쿼리를 처리하는데 한계점이 존재한다.

`이러한 문제들을 해결해주는것이 Querydsl`

`Querydsl`
- 쿼리를 문자가 아닌 자바 코드로 작성
- 문법 오류를 컴파일 시점에 잡아준다.
- 동적 쿼리 문제 해결
- IDE 도움을 받을 수 있음

```java
List<Membmer> members = queryFactory
    .select(member)
    .from(member)
    .where(member.username.eq(username))
    .fetch();
```

> Querydsl은 선택이 아닌 필수!
