# 실전! 스프링 데이터 JPA - 공통 인터페이스 설정

#### 공통 인터페이스 설정
```java
@SpringBootApplication
// SpringBoot 이기 때문에 생략이 가능하다.
//@EnableJpaRepositories(basePackages = "study.datajpa.repository")
public class DataJpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(DataJpaApplication.class, args);
    }

}
```
- JavaConfig 설정 - 스프링부트 사용시 생략이 가능하다.
    - 만약 경로가 달라진다면, 설정이 필요함

MemberJpaRepository는 구현코드가 모두 있지만, MemberRepository를 살펴보면 인터페이스만 존재한다.
- JpaRepository를 상속받고 있는 구조
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

}
```

- 주입받은 MemberRepository 를 출력해보면 다음과 같다.
    - class com.sun.proxy.$Proxy...

- @Repository 애노테이션을 생략 가능하다.
    - 자동으로 빈으로 등록이 된다.
    - JPA 예외를 스프링 공통 예외로 처리

> org.springframework.data.repository.Repository를 구현한 클래스는 스캔의 대상이 된다.

> 개발자는 인터페이스만 선언해주면, Spring data JPA가 애플리케이션 로딩시점에 구현체를 만들어서 Injecting 해준다.
