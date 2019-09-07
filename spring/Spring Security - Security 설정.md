# Spring Security - Security 설정하기
- 스프링 시큐리티 설정을 적용한다.
- 부트가 제공하는 자동설정 외에 커스터마이징을 진행한다.

#### 설정파일
- config/SecurityConfig 클래스를 생성한다.
- @Configuration 애노테이션으로 설정 클래스 임을 스프링에게 알려준다.
- @EnableWebSecurity
    - WebSecurity 적용을 알린다.
    - 기존의 스프링부트가 제공하던 자동설정들은 모두 무시 된다.
- WebSecurityConfigurerAdapter
    - 스프링 시큐리티 설정을 편리하게 하기위한 클래스이다.

시큐리티 설정은 주로 WebSecurityConfigurerAdapter 의 Method를 Override 하는 방식으로 한다.
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
    }
}
```

#### 시큐리티 설정
- HttpSecurity를 활용하여 설정을 진행한다.
- http.authorizeRequest() 메서드를 사용하여 요청 URI에 인증 관련 설정을 진행한다.
- .mvcMatchers("/", "/info").permitAll() 
    - / , /info 는 인증이 되지 않아도 접근이 가능하도록 한다.
- .mvcMatchers("/admin").hasRole("ADMIN")
    - /admin 은 ADIMN 권한이 필요하다.
- .anyRequest().authenticated()
    - 그 외 모든 요청은 인증이 필요하다
- .formLogin()
    - 폼 인증 방식을 적용한다.
- .httpBasic()
    - httpBasic 인증 방식을 사용한다.
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/", "/info").permitAll()
                .mvcMatchers("/admin").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .and()
            .httpBasic();
    }
}
```

#### 해결된 문제
- 요청 URI 별로 인증이 필요하지 않을경우 해당 설정이 적용가능하다.
- /admin은 ADMIN 권한이 있는 경우에만 접근이 가능하다.

#### 해결되지 않은 문제
- 여전히 기본 유저가 하나 생성 가능하다.
- 새로운 유저가 생성이 불가능 하다.
- password가 log에 남는다.
