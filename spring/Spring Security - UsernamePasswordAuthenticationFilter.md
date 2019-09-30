# Spring Security - UsernamePasswordAuthenticationFilter
- Spring Security 에서 Form Login 인증을 처리하는 필터이다.
- 이전에 Spring Security 아키텍쳐를 살펴볼때 디버깅을 통해 살펴보았던 Filter이다.

#### UsernamePasswordAuthenticationFilter
- username, pasword로 Authentication객체를 생성하고, AuthenticaionManager를 사용하여 인증을 시도한다.
- 여러개의 AuthenticationProvider를 사용하여 인증을하는데 그중에서도 **DaoAuthenticationProvider**를 사용한다.
- DaoAuthenticationProvider는 **UserDetailsService**를 사용하여 인증을 시도하는데, 이 객체가 바로 우리가 구현한 UserDetailsService이다.
- AuthenticationProvider는 Parent 를 가지고있으며 현재 Provider가 처리를 하지 못한다면, 부모에게로 가 요청을 처리하는 식의 계층구조로 되어있다.

https://pupupee9.tistory.com/104?category=796566
