# 실전! 스프링 데이터 JPA - Projections

#### Projections
- https://docs.spring.io/spring-data/jpa/docs/2.2.5.RELEASE/reference/html/#projections
- DB에서 엔티티 대신 DTO를 편리하게 조회할 때 사용한다.
- 전체 엔티티가 아닌 회원의 이름만 조회하고 싶다면 ?

> Projections를 쉽게 비유하면, Query의 select절에 명시하는 필드들이라고 생각하면 된다.

`ClosedProjections`
```java
public interface UserNameOnly {

    // close projection은 필요한것만 정확하게 가져오는것
    String getUsername();

}
public interface MemberRepository {
    List<UserNameOnly> findProjectionsByUsername(@Param("username") String username);
}

@Test
public void projections() {
    List<UserNameOnly> result = memberRepository.findProjectionsByUsername("m1");
    // JDKDynamicProxy 객체를 담고 있음.
    // 인터페이스를 정의하면 실제 구현체는 Spring data JPA가 만들어 줌
    for (UserNameOnly userNameOnly : result) {
        System.out.println("userNameOnly = " + userNameOnly.getUsername());
    }
}
```
- Proxy기반으로 동작한다.
    - 실체 구현체는 Spring data JPA가 생성
- interface에서 getter 형태로 정의한 필드들만 SELECT절을 이용해 정확하게 가져온다.


`OpenProjections`
```java
public interface UserNameOnly {

    // SpEL 을 사용
    // Member Entity의 Property를 모두 조회 한뒤 SpEL에 명시한 데이터를 조합
    // DB에서 모두 조회한뒤 처리 하는것 open projection
    @Value("#{target.username + ' ' + target.age}")
    String getUsername();
}
```
- Proxy기반으로 동작한다.
    - 실체 구현체는 Spring data JPA가 생성
- SpEL을 지원한다.
- Member Entity의 모든 필드를 조회 (엔티티를 조회) 한뒤 SpEL에 명시한 데이터를 조합한다.

> DB에서 모든 필드를 조회한뒤, 애플리케이션에서 처리한다.

`ClassProjections`
```java
public class UsernameOnlyDto {

    private final String username;

    // 생성자의 파라메터명과 매칭시켜 Projection이 동작하낟.
    public UsernameOnlyDto(String username) {
        this.username = username;
    }

    public String getUsername() {
        return username;
    }
}

@Test
public void projections() {
    // class Projection
    // Proxy를 사용하지 않고, 실제 구현체인 Class를 사용한다.
    List<UsernameOnlyDto> results = memberRepository.findClazzProjectionsByUsername("m1");
    for (UsernameOnlyDto usernameOnlyDto : results) {
        System.out.println("usernameOnlyDto = " + usernameOnlyDto);
    } 
}
```
- Proxy를 사용하지 않고 Class를 사용한다.
- 생성자의 파라메터 이름과 매칭시켜 Projections가 동작하게 된다.

`Dynamic Projections`
```java
public interface MemberRepository {
    <T> List<T> findClazzDynamicProjectionsByUsername(@Param("username") String username, Class<T> type);  
}

@Test
public void projections() {
    // dynamic Projection
    // 동적 프로잭션
    List<UsernameOnlyDto> results2 = memberRepository.findClazzDynamicProjectionsByUsername("m1", UsernameOnlyDto.class);
    for (UsernameOnlyDto usernameOnlyDto : results2) {
        System.out.println("usernameOnlyDto = " + usernameOnlyDto);
    }
}
```
- 제네릭을 이용해 쿼리는 동일하지만, 매번 다른 Projections를 사용이 가능하도록 지원한다.

`중첩구조 Projections`
```java
// Username, Team의 이름을 가져옴
public interface NestedClosedProjections {

    String getUsername();
    TeamInfo getTeam();

    interface TeamInfo {
        String getName();
    }
}

@Test
public void projections() {
    // 중첩구조 Projection
    // Member는 Username은 정확하게 최적화가 된다 (Root)
    // 2뎁스 부터는 최적화가 되지않고 Team 엔티티를 가져온다.
    // join은 leftJoin으로 들어간다.
    List<NestedClosedProjections> results3 = memberRepository.findClazzDynamicProjectionsByUsername("m1", NestedClosedProjections.class);
    for (NestedClosedProjections nestedClosedProjections : results3) {
        String username = nestedClosedProjections.getUsername();
        String teamName = nestedClosedProjections.getTeam().getName();
        System.out.println("username = " + username);
        System.out.println("teamName = " + teamName);
    }    
}
```
- Projections의 최상위를 Root 라고 한다.
- 2뎁스인 TeamInfo 부터는 SELECT절 최적화가 되지않고, Team 엔티티 전체를 가져온다.

`주의점`
- 프로젝션 대상이 Root 엔티티라면 최적화 가능
- 프로젝션 대상이 Root가 아닐 경우
    - LEFT OUTER JOIN 처리
    - 모든필드를 SELECT해서 엔티티로 조회한 뒤 계산

#### 정리
- 프로젝션 대상이 Root 엔티티라면 유용
- 프로젝션 대상이 Root 엔티티를 넘어가면 SELECT 최적화가 불가능
- 실무의 복잡한 쿼리를 해결하기에는 한계가 있음
- 단순할 경우에만 사용하고, 복잡해지면 QueryDSL을 사용
