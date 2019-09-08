# Spring Security - JPA 연동하기
- InMemoryUser 방식은 문제점이 존재하기때문에 DATABASE와 연동하는 방법을 사용한다.
- 방법이 다양하지만 그중에서도 JPA와 연동하는 방법을 사용한다.

#### 의존성 추가하기
- JPA 의존성을 추가하고, DATABASE는 h2 를 사용하도록 한다.
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<scope>runtime</scope>
</dependency>
```

#### Account
- User 정보에 해당하는 Account Entity 를 생성한다.
- username은 유일한 값이기 때문에 Unique 제약조건을 걸어준다.
```java
@Entity
@Getter @Setter
public class Account {

    @Id @GeneratedValue
    private Integer id;

    @Column(unique = true)
    private String username;

    private String password;

    private String role;
}
```

- JpaRepository 를 상속받는 AccountRepository 를 생성해준다.
- 기본적인 CRUD, Paging 관련 구현이 모두 끝난다.
- 또한 추가적으로 username으로 조회를 해야하기때문에 username으로 조회하는 namedQuery Method를 정의해준다.
```java
public interface AccountRepository extends JpaRepository<Account, Integer> {

    Account findByUsername(String username);
}
```

#### UserDetailsService
- Spring Security 에서 User 정보를 Database와 연동하여 가져오는 역할을 담당하는 인터페이스 이다.
- 이에 대한 구현체는 NoSQL, RDBMS 등 어떤 방식으로 구현을 하던 제약사항이 없다.
- 그저 Username에 해당하는 User를 조회해 UserDetails Type으로 리턴해주는 역할만 한다.

- 객체지향 원칙중 하나인 단일 책임의 원칙에 의하면 하나의 역할만 하는것이 맞지만 UserDetailsService의 역할도 AccountService의 역할중 하나 이기때문에 UserDetailsService의 구현체로 사용한다.

- loadUserByUsername(String username) 의 구현
	- username에 해당하는 유저정보를 DB로 부터 읽어온다.
	- 해당하는 유저가없다면 UsernameNotFoundException 예외를 발생시킨다.
	- 존재한다면 UserDetails 를 구현하는 User Type으로 Return 해준다
		- 이전에는 User라는 클래스를 제공해 주지않아 이를 연결해주는 Adapter 클래스를 많이 사용하였다.
		- 하지만 상황에따라 Adapter 클래스를 사용할때도 존재한다.
```java
@Service
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    private final AccountRepository accountRepository;

    // Database에서 조회해서 UserDetails Type으로 Return 해줘야함
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account account = accountRepository.findByUsername(username);
        if (account == null) {
            throw new UsernameNotFoundException(username);
        }
        return User.builder()
                .username(account.getUsername())
                .password(account.getPassword())
                .roles(account.getRole())
                .build();
    }
}
```

#### SecurityConfig
- 기존의 InMemory 설정은 더이상 필요가 없어졌기때문에 해당 설정은 다음과 같이 제거해준다.
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

    /*
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("admin").password("{noop}1234").roles("ADMIN").and()
                .withUser("user").password("{noop}user1234").roles("USER");
    }
     */
}
```

#### 문제점
- 현재는 애플리케이션을 실행하고나면 User를 생성할 방법이 없다.
- 따라서 편의상 Account를 생성할수 있도록 AccountController클래스를 생성해야한다.

#### AccountController
- PathVariable로 role/username/password를 받아 유저를 생성하고, 생성된 유저를 반환해주는 RestController를 구현한다.
- 하지만 이 방법은 절대 사용해선 안된다.
- 편의상 구현한 방법이지 절대 사용해선 안되는 방법이다.
```java
@RestController
@RequiredArgsConstructor
public class AccountController {

    private final AccountRepository accountRepository;

    @GetMapping("/account/{role}/{username}/{password}")
    public Account createAccount (@ModelAttribute Account account) {
        return accountRepository.save(account);
    }
}
```

#### 또 다른 문제점
- 유저를 생성할수 있도록 AccountController를 생성해주었는데 무엇이 문제일까 ?
- 바로 Account를 생성하는 url인 /account/** 에 접근하지 못한다.
- Security 설정에 의해 현재 상태에서는 인증을 할수 없기때문...

다음과 같이 SecurityConfig에서 /account/** 를 허용해준다.
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/", "/info", "/account/**").permitAll()
                .mvcMatchers("/admin").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .and()
            .httpBasic();
    }

    /*
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("admin").password("{noop}1234").roles("ADMIN").and()
                .withUser("user").password("{noop}user1234").roles("USER");
    }
     */
}
```

#### 유저 생성해보기
- http://localhost:8080/account/USER/user/1234 로 요청을 보내면 해당 유저가 생성되고, 다음과 같이 응답을 받는다.
```javascript
{"id":1,"username":"user","password":"1234","role":"USER"}
```

#### 생성된 유저로 접근하기
```javascript
{"id":1,"username":"user","password":"1234","role":"USER"}
```

위와 같이 생성된 유저정보를 활용하여 접근을 해보도록 하자
- /dashboard 로 접근하면 시큐리티 기본 로그인폼이 나타나고, user/1234로 로그인을 시도하면 로그인이 되지않는다.
- 다음과 같은 예외가 발생한다.
	- 이전에서 설명한 {noop}1234 형식의 패스워드가 아니기때문에 발생한 문제이다.
```java
java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
```


#### 해결방법
- 먼저 AccountController의 코드를 수정한다.
- 패스워드를 인코딩하는등의 로직은 Service 계층에 있는것이 맞기때문에 AccountService로 의존성을 교체해준다.
- 또한 AccountService의 createAccount Method로 유저를 생성하도록 변경한다.

* AccountController
```java
@RestController
@RequiredArgsConstructor
public class AccountController {

    private final AccountService accountService;

    @GetMapping("/account/{role}/{username}/{password}")
    public Account createAccount (@ModelAttribute Account account) {
        return accountService.createAccount(account);
    }
}
```

- AccountService의 createAccount Method이다.
- 생성할 Account 를 받아 password Encoding을 진행한다.
	- {noop}1234 의 형식
* AccountService
```java
public Account createAccount(Account account) {
	account.encodePassword();
	return accountRepository.save(account);
}
```

- 정상적인 passwordEncoder 를 사용한 방법은 아니지만 패스워드를 평문으로 저장하기때문에 굳이 passwordEncoder를 사용하지않고,
- Security에서 인식하도록 형식만 맞춰주도록 구현한다.
- passwordEncoding 로직은 Service 계층에 존재할수도 있지만. Account Domain과 관련된 로직이기 때문에 Account Entity 클래스에 정의해 주었다.
* Account
```java
@Entity
@Getter @Setter
public class Account {

    @Id @GeneratedValue
    private Integer id;

    @Column(unique = true)
    private String username;

    private String password;

    private String role;

    public void encodePassword() {
        this.password = "{noop}" + this.password;
    }
}
```

* 애플리케이션 을 재기동 하고,  유저를 생성한뒤 /dashboard로 접근 해보자.
다음과 같이 유저가 생성되며, 정상적으로 /dashboard로 접근이 가능해진다.
```java
{"id":1,"username":"user","password":"{noop}1234","role":"USER"}
```


* SecurityConfig 살펴보기
- 우리는 UserDetailsService의 구현체인 AccountService를 구현했지만 이를 Spring Security에 설정하지 않았는데도 AccountService를 사용하고있다.

- 다음과 같이 명시적으로 UserDetailsService를 사용하라고 명시해 주어야 한다.
- 하지만 UserDetailsService 타입의 Bean이 등록되어 있다면 이런 과정을 생략할 수 있다.
- 자동적으로 userDetailsService로 설정한다.
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    AccountService accountService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(accountService);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/", "/info", "/account/**").permitAll()
                .mvcMatchers("/admin").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .and()
            .httpBasic();
    }

    /*
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("admin").password("{noop}1234").roles("ADMIN").and()
                .withUser("user").password("{noop}user1234").roles("USER");
    }
     */
}
```
