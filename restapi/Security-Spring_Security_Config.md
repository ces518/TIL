# RESTAPI - Security - Spring Security 기본설정
- 스프링 시큐리티가 의존성에 존재하면 시큐리티 자동 설정이 동작
    - 모든 요청에 인증이 필요하게됨.
        - 모든 컨트롤러 테스트는 실패하게 됨.
    - 시큐리티가 사용자를 인메모리로 임의로 생성해준다.

#### 시큐리티 설정
- 시큐리티 필터 미적용
    - /docs/index.html
- 인증단계 없이 접근
    - GET /api/events
    - GET /api/events/{id}
- 인증이 필요
    - 나머지 모든 API 들
    - POST /api/events
    - PUT /api/events/{id}
    - ...


#### Spring Security OAuth2.0 설정
- 공통적으로 필요한 설정 적용하기

- PasswordEncoder Bean 등록
    - AppConfig 클래스를 생성하여 Application 설정들을 모아놓음 
```java
@Configuration
public class AppConfig {

    /**
     * 스프링 시큐리티 최신버전에 들어간 패스워드인코더
     * 인코딩 패스워드 문자열 앞에 어떤 방식으로 인코딩이 된건지 prefix 를 적용한다.
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder () {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

}
```

- Security 설정
    - SecurityConfig 클래스 생성
        - @EnableWebSecurity 애노테이션을 사용하는 순간 SpringBoot 가 제공하는 기본 시큐리티 설정은 적용되지않음.
        - WebSecurityConfigurerAdapter 클래스를 상속받아 시큐리티 설정 진행
    - Security 설정에 필요한 의존성 주입
        - AccountService
        - PasswordEncoder
    - TokenStore Bean 생성
        - OAuth Token 을 저장할 저장소
        - InMemoryTokenStore
    - authenticationManager를 다른 서버가 참조할 수 있도록 Bean으로 등록해준다.
        - Resource-Server
        - AuthorizationService

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    /* UserDetailsService */
    @Autowired
    AccountService accountService;

    @Autowired
    PasswordEncoder passwordEncoder;

    @Bean
    public TokenStore tokenStore () {
        return new InMemoryTokenStore();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(accountService)
                .passwordEncoder(passwordEncoder);
    }

    /*
        정적인 리소스들은 필터적용 무시
        Filter Chain 을 타지않고 Filter 이전에 걸러버림
    */
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().mvcMatchers("/doc/index.html");
        web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }

    /* http 로 거르면 filter로 들어온 상태
        Security Filter Chain 을 탄다.
    * */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/docs/index.html").anonymous()
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations()).anonymous()
        ;
    }
}
```

#### WebSecurity vs HttpSecurity
- Security 인증을 적용할지 여부를 설정할수 있는 클래스가 두가지이다.
- HttpSecurity
    - Security Filter 로 들어온 이후에 Role 등을 이용하여 인증 적용 여부를 결정한다.
    - Security Filter Chain 을 사용함
- WebSecurity
    - Filter 를 타기 이전에 무시해버린다.
    - Security Filter Chain 을 사용하지 않기떄문에 서버에서 처리하는 양이 줄어듦
