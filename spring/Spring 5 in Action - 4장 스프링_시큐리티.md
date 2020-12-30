# Spring 5 in Action

## 4장 스프링 시큐리티

### 스프링 시큐리티 활성화
- 스프링 시큐리티를 사용하기 위해서는 spring-boot-starter-security 의존성을 추가해 주어야 한다.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- 의존성을 추가한 뒤 애플리케이션을 실행하면, 스프링부트에서 제공하는 시큐리티 자동설정이 적용된다.
- 모든 페이지에 인증 이 필요하게 되고 시큐리티에서 기본 제공하는 로그인 폼을 통해 인증을 해야 리소스에 접근이 가능해진다.
- 이때 인증에 필요한 계정이 필요하게 되는데, username: USER, password 는 매번 실행때 마다 새롭게 generate 된다.
  - UserDetailsServiceAutoConfiguration 클래스가 이를 수행한다.
- 애플리케이션 실행 로그를 살펴보면 **Using generated security password:** 로 시작하는 로그를 확인하자.

### 스프링 시큐리티 설정
- 스프링 시큐리티가 제공하는 기본설정만으로는 부족한 점이 많다.
- 별도의 설정을 통해 시큐리티 설정을 추가적으로 해줘야만 비로서 쓸만하게 (?) 된다.

`SecurityConfig`
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/design", "/orders").hasRole("USER")
                .antMatchers("/", "/**").permitAll()
            .and()
                .httpBasic()
        ;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        /*
            InMemory 기반 사용자 스토어
        */
        auth.inMemoryAuthentication()
                .withUser("user1")
                .password("{noop}password1")
                .authorities("ROLE_USER")
                .and()
                .withUser("user2")
                .password("{noop}password2")
                .authorities("ROLE_USER");
    }
}
```
- **@EnableWebSecurity** 애노테이션 을 사용해 WebSecurity 설정임을 선언한다.
  - 이 애노테이션을 선언하게 되면, 스프링부트가 제공하는 자동 설정은 **모두 무시** 된다. (자동 설정을 쓰는경우는 거의 없다..)
- WebSecurityConfigurerAdapter 클래스를 상속 받고 있다.
  - 이는 스프링시큐리티 설정을 편리하게 하기위한 클래스이다. (a.k.a. WebMvcConfigurerAdapter)
- configure(http) 를 통해 **인가** 설정을 한다.
  - http.authorizeRequest() 메서드를 사용하여 요청 URI에 인증 관련 설정을 진행한다.
  - .antMatchers("/design", "/orders").hasRole("USER")
    - /design, /orders 에 접근하려면, USER 권한 (ROLE_USER) 이 있어야 한다.
  - .antMatchers("/", "/**").permitAll()
    - 그외는 인증이 필요하지 않다.
  - .httpBasic()
    - **HTTP BASIC** 인증 방식을 사용한다.
- configure(auth) 를 통해 사용자 스토어 설정을 한다.
  - 위는 인메모리 사용자 스토어 이며, 스프링 시큐리티에서는 여러가지 사용자 스토어 구현방법을 제공한다.
  - JDBC, LDAP .. 등등

`HTTP BASIC`
- HTTP 스펙 중 Header에 username, password 를 실어 보내면 브라우저 혹은 서버가 그 값을 읽어 인증하는 방식이다.
- 보통 브라우저 기반 요청이 클라이언트의 요청을 처리할때 자주 사용한다.
- 보안에 상당히 취약하기 때문에 반드시 HTTPS를 사용할 것을 권장한다.
  - 요청이 하나라도 Snipping 당한다면 인증정보가 노출된다.
- https://en.wikipedia.org/wiki/Basic_access_authentication
  
#### JDBC 사용자 스토어 사용시 Security 스키마
`schema.sql`
```sql
drop table if exists users;
drop table if exists authorities;
drop index if exists ix_auth_username;

create table if not exists users (
    username varchar2(50) not null primary key,
    password varchar2(50) not null,
    enabled char(1) default '1'
);

create table if not exists authorities (
    username varchar2(50) not null,
    authority varchar2(50) not null,
    constraint fk_authorities_users foreign key(username) references users(username)
);

create unique index ix_auth_username on authorities (username, authority);
```

`data.sql`
```sql
insert into users (username, password) values ('user1', 'password1');
insert into users (username, password) values ('user2', 'password2');

insert into authorities (username, authority) values ('user1', 'ROLE_USER');
insert into authorities (username, authority) values ('user2', 'ROLE_USER');
```

### JPA 와 Security 연동하기
- 보통 사용자 데이터도 관계형 데이터베이스에 저장을 하게 될텐데, 이 또한 타코 데이터와 마찬가지로 JPA 를 사용하도록 변경한다.

#### 사용자 도메인 객체와 퍼시스턴스 정의
`User`
```java
@Entity
@Data
@NoArgsConstructor(access = AccessLevel.PROTECTED, force = true)
@RequiredArgsConstructor
public class User implements UserDetails {
    private static final long serialVersionUID = 1L;

    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private final String username;
    private final String password;
    private final String fullname;
    private final String street;
    private final String city;
    private final String state;
    private final String zip;
    private final String phoneNumber;


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("ROLE_USER"));
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```
- Spring Security에서 제공하는 UserDetails 인터페이스를 구현하도록 한다.
- Spring Security 에서 사용자 정보를 가진 객체는 UserDetails 타입이여야 한다. 
  - 도메인 객체가 이를 직접구현하는 것이 베스트프렉티스는 아니다. 
  - Adapter 클래스를 사용하는 것이 좀 더 좋다.
- 또한 시큐리티에서는 username 으로 조회가 필요하기 때문에 다음과 같이 UserJpaRepository 를 작성한다.

`UserJpaRepository`
```java
public interface UserJpaRepository extends CrudRepository<User, Long> {
    User findByUsername(String username);
}
```

#### 사용자 명세 서비스
- Spring Security 와 JPA 를 연동하려면 **UserDetailsService** 를 구현해야 한다.

`UserDetailsService`
```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```
- 위에서 사용자 도메인 객체를 정의할때 왜 UserDetails 타입을 구현했는지, 이유가 여기에 있다.
- Spring Security 는 사용자 스토어에서 유저를 조회할때 UserDetailsService 를 사용하는데, 반환하는 타입이 UserDetails 타입이여야만 한다.

`UserRepositoryUserDetailsService`
```java
@Service
public class UserRepositoryUserDetailsService implements UserDetailsService {

  @Autowired
  private UserJpaRepository userRepository;

  @Override
  public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userRepository.findByUsername(username);
    if (user != null) {
      return user;
    }
    throw new UsernameNotFoundException(String.format("User '%s' not found", username));
  }
}
```

#### 사용자 명세 서비스와 Security 연동
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService userDetailsService;
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(encoder())
        ;
    }

    @Bean
    public PasswordEncoder encoder() {
        return new BCryptPasswordEncoder();
    }
}
```
- 앞에서 작성한 UserDetailsService 의 구현체를 주입받아 설정을 해주고, **PasswordEncoder** 를 추가로 설정해 주었다.
- PasswordEncoder 는 패스워드를 암호화에 대한 인터페이스 이다.
  - 구현체는 여러가지가 있다.
  - NoOpPasswordEncoder
  - BCryptPasswordEncoder
  - 등등...
- Spring Security 5버전에 들어서면서, **PasswordFormat** 이 추가되었다.

`PasswordFormat 이 생긴 이유?`
- 이전 버전에서는 기본 전략이 NoOpPasswordEncoder 였다.
- Security 5 버전으로 올라오면서 BCrypt 로 변경되었다.
- 이로서 문제가 발생!!
  - 이전의 버전을 사용하던 사람들은 평문으로 사용했을 수도 있다.
  - 다른 암호화 방식을 사용하고 싶은 사람도 있다.
- 다양한 패스워드 방식을 제공하기위해 format이 변경되었다.

`SpringSecurity 5.x 기본 PasswordEncoder`
- sha256, scrypt 등 id에 해당하는 알고리즘 명을 주고, 실제 패스워드 인코더 객체를 설정해주면, 알고리즘에 해당하는 패스워드 인코드를 사용한다.
- SpringSecurity 5 에서 기본 패스워드 인코더로 사용하는 bcrypt 방식을 암호화를 사용하는 PasswordEncoder를 사용해보자

* Bcrypt PasswordEncoder를 Bean으로 등록
```java
@Bean
public PasswordEncoder passwordEncoder () {
	return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```

- https://java.ihoney.pe.kr/498

### 웹 요청 보안 설정
- 앞에서 잠깐 살펴보았지만, SecurityConfig 클래스의 configure(http) 메소드를 구현해서 웹 요청 보안 설정을 할 수 있다.
- 이는 HttpSecurity 객체를 인자로 받는데, 이를 사용해 구성할 수 있는것은 다음과 같다.
1. HTTP 요청을 허용하기 전에 만족해야할 보안 조건 구성
2. 커스텀 로그인 페이지 구성
3. 로그아웃
4. CSRF 방어

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .antMatchers("/design", "/orders").hasRole("USER")
            .antMatchers("/", "/**").permitAll()
    ;
}
```
- 위에서 잠깐 살펴보았던 설정의 일부이다.
- /design, /orders 를 ROLE_USER 권한을 가진 사용자에게만 허용하며, 이외 모든 요청은 모든 사용자에게 허용된다.
- 이런 규칙을 설정할 때는 순서가 중요한데, permitAll 설정이 먼저 온다면, /design, /orders 설정은 효력이 사라진다.

`보안 처리 방법을 정의하는 메소드 목록`

| 메소드 | 설명 |
| --- | --- |
| access(String) | 인자로 전달된 SpEL 표현식이 참이라면 접근 허용 |
| anonymous() | 익명 사용자에게 접근 허용 |
| authenticated() | 인증된 사용제에게 접근 허용 |
| denyAll() | 무조건 접근 거부 |
| fullyAuthenticated() | 익명이 아니거나 remember-me 가 아닌 사용자로 인증되면 접근 허용 |
| hasAnyAuthority(String..) | 지정된 권한중 어떤 것이라도 있다면 접근 허용 |
| hasAnyRole(String..) | 지정된 역할 중 어느 하나라도 갖고 있다면 접근 허용 |
| hasAuthority(String) | 지정된 권한을 사용자가 갖고 있다면 접근 허용 |
| hasIpAddress(String) | 지정된 IP 로 요청이 오면 접근 허용 |
| hasRole(String) | 지정된 역할을 갖고 있다면 접근 허용 |
| not() | 다른 접근 메소드들의 효력을 무효화 |
| permitAll() | 무조건 접근 허용 |
| rememberMe() | remember-me 를 통해 인증된 사용자 접근허용 |

#### CSRF 방어
- **CSRF (Cross Site Request Forgery)** 는 많이 알려진 보안 공격이다.
- 사용자가 로그인한 웹사이트를 이용해 요청 위조하여 공격 대상이 되는 사이트에 폼이 제출되도록 하는 방식이다.
- CSRF 공격을 막기 위해 애플리케이션에는 폼의 hidden 필드로 CSRF 토큰을 생성해서 이를 이용해 유효한 요청인지를 검증한다.
- 스프링 시큐리티에는 내장된 CSRF 기능이 있으며 이는 기본적으로 활성화 되어있다.
- 절대 비활성화 하지 말아야하며, REST API 서버라면 이는 비활성화 해주어야만 한다.

### 사용자 인지하기
- 사용자가 로그인이 되었음을 아는것에 그치지않고 사용자 경험에 맞추려면 사용자에 대한 정보도 필요한 때가 있다.
- 스프링 시큐리티에서 사용자 정보를 사용하는 방법은 여러가지가 있고, 가장 많이 사용되는 방법은 아래 4가지 이다.
1. Principal - 컨트롤러 파라미터로 주입
2. Authentication - 컨트롤러 파라미터로 주입
3. SecurityContextHolder - 시큐리티 컨텍스트를 얻어 사용
4. @AuthenticationPrincipal - 애노테이션을 통해 컨트롤러 파라미터로 주입

- Principal 객체와 @AuthenticationPrincipal 의 가장 큰 차이는 어느정도 수준까지의 정보를 참조 가능하냐 이다.
  - Principal 은 JAVA 표준 Principal 객체이며 우리가 참조할수 있는 정보는 name 정보 밖에 없다.
  - @AuthenticationPrincipal 은 UserDetailsService 에서 리턴하는 객체를 직접 사용할 수 있다.
- 또한 SecurityContextHolder 를 통해 얻는 Principal 객체와는 다르다는 점을 인지해야 한다.

## 정리
- 스프링 시큐리티의 자동 구성은 실무에서 사용하기엔 힘들기 때문에 별도의 설정이 반드시 필요하다.
- 시큐리티에서 지원하는 사용자 스토어는 RDB, LDAP 등을 지원한다.
- 시큐리티에서는 자동으로 CSRF 공격을 방어한다.
- 인증된 사용자 에 대한 정보를 얻기 위해 Principal, Authentication, SecurityContextHolder, @AuthenticationPrincipal 를 사용 할 수 있다.
