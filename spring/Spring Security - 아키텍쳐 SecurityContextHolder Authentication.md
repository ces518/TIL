# Spring Security - ArcheTecher_SecurityContextHolder Authentication
#### SecurityContextHolder
- Security Context 제공, 기본적으로 ThreadLocal을 사용한다.
- 기본 전략은 ThreadLocal 기반으로 동작한다.
- SecurityContextHolder를 통해 SecurityContext에 접근할 수 있다.

#### SecurityContext
- Authentication을 제공한다.

#### Authentication
- Spring Security에서 인증을 거치고난 뒤의 사용자의 인증정보를 Principal 이라고 한다.
- 이러한 Principal을 감싸고 있는 객체이다.
- Printcipal 외에도 GrantedAuthorities (사용자 권한 정보) 등 다양한 정보를 가지고 있다.
- SecurityContext 를 통해 Authentication 객체를 받아올 수 있다.

#### Principal
- UserDetailsService에서 리턴한 객체이다.
- UserDetails 타입
- 인증에 성공한 사용자 정보에 해당한다.

#### GrantedAuthorities
- ROLE_USER, ROLE_ADMIN 등 Principal이 가지고 있는 권한을 나타낸다.
- 인증 이후 인가 및 권한을 확인할때 참조한다.

#### ThreadLocal
- ThreadLocal은 같은 Thread 내에서 공유하는 저장소이다.
- Parameter를 넘기지 않아도, 쓰레드 내에서 데이터를 공유 및 전달이 가능하다.

#### 사용 예제
```java
@Service
public class SampleService {

    public void dashboard () {
        // SecurityContextHolder를 통해 SecurityContext에 접근이 가능하다.
        SecurityContext context = SecurityContextHolder.getContext();

        // SecurityContext 에서 Authentication 객체를 얻는다.
        Authentication authentication = context.getAuthentication();

        // Authentication 객채에서 사용자 인증정보인 Principal (UserDetailsService에서 리턴해준 타입 -> UserDetails Type)
        Object principal = authentication.getPrincipal();

        // Authorities 사용자의 권한을 의미 하는 객체 이다. User 객체를 만들때 roles 에 추가한 권한 객체 들이다.
        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();

        // Holder가 Authentication을 담고있음
        // ThreadLocal 을 사용한다.
        // 애플리케이션 어디서든 접근이 가능하다.
        // 처리하는 Thread가 달라진다면, 제대로된 Authentication 정보를 받아오지 못한다.
    }
}
```

참조
- https://docs.spring.io/spring-security/site/docs/5.1.5.RELEASE/reference/htmlsingle/#core-components
