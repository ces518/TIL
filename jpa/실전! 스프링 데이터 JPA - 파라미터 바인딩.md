# 실전! 스프링 데이터 JPA - 파라미터 바인딩

#### 파라미터 바인딩
- 위치기반, 이름 기반 방식을 제공한다.
- 위치기반은 절대 사용하지말고 이름 기반 방식을 사용할것

`파라미터 바인딩`
```java
@Query("select m from Member m where m.username = :username and m.age = :age")
List<Member> findUser(@Param("username") String username, @Param("age") int age);
```

> 코드 가독성과 유지보수를 위해 이름 기반 파라미터 바인딩을 사용할것

`컬렉션 파라미터 바인딩`
- Collection 타입으로 in 절 지원

```java
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") List<String> names);
```
