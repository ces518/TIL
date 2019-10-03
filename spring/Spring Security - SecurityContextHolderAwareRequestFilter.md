# Spring Security - SecurityContextHolderAwareRequestFilter
- Spring Security 에서 시큐리티 관련 서블릿 3 API 를 구현해주는 필터이다.
- 딱히 우리가 신경쓸 일은 없는 필터이다.

#### SecurityContextHolderAwareRequestFilter
- Spring Security 에서 서블릿 3 스펙을 지원한다.
- 시큐리티 관련 메서드들을 스프링 시큐리티 기반으로 구현체를 연결해주는 역할을 한다.
- HttpServletRequest authenticate();
    - 인증이 된건지 아닌건지 판단 안되어있을경우 로그인 페이지로 이동 한다.
- HttpServletRequest login();
    - AuthenticationManager를 사용하여 인증을 처리하도록 한다.
- HttpServletRequest logout();
    - LogoutHandler를 사용하여 로그아웃을 진행한다.
- AsyncContext start();
    - SecurityContext를 지원한다.
    - 새로이 생성한 스레드에도 SecurityContext가 공유되도록 하는역할
    - WebAsyncIntergractionFilter와 비슷하다.

![SecurityContextHolderAwareRequestFilter](./images/SecurityContextHolderWareRequestFilter.png)
