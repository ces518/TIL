# REST API - Event API 점검

#### ResourceServer
- 기존 리소스 서버의 설정에 잘못된 부분이 존재
```java
@Override
public void configure(HttpSecurity http) throws Exception {
    http
        .anonymous()
            .and()
        .authorizeRequests()
            .mvcMatchers(HttpMethod.GET, "/api/**").anonymous()
            .anyRequest().authenticated()
            .and()
        .exceptionHandling()
            .accessDeniedHandler(new OAuth2AccessDeniedHandler())
        // 인증이 안되거나, 권한이없는경우 예외가 발생하며 OAuth2AccessDeniedHandler 가 403 응답을 내보낸다.
    ;
}
```

anonymous 만 허용하도록 설정을 하였기때문에 인증을 한 사용자만 접근이 가능하다.
우리가 의도한 방식의 API는 GET 요청은 모두 인증 필요없이 접근이 가능해야한다.
기존의 설정을 permitAll로 변경하자.
```java
mvcMatchers(HttpMethod.GET, "/api/**").anonymous()
```

#### 변경된 ResourceServer 설정
- GET 요청은 인증 절차 필요없이 리소스에 접근이 가능해진다.
```java
@Override
public void configure(HttpSecurity http) throws Exception {
    http
        .anonymous()
            .and()
        .authorizeRequests()
            .mvcMatchers(HttpMethod.GET, "/api/**").permitAll()
            .anyRequest().authenticated()
            .and()
        .exceptionHandling()
            .accessDeniedHandler(new OAuth2AccessDeniedHandler())
        // 인증이 안되거나, 권한이없는경우 예외가 발생하며 OAuth2AccessDeniedHandler 가 403 응답을 내보낸다.
    ;
}
```

#### 추가 기능 정리
- 이벤트 목록 API
    - 로그인 했을때
        - 이벤트 생성 링크 제공
- 이벤트 조회
    - 로그인 했을때
        - 이벤트 Manager인 경우 이벤트 수정 링크 제공
