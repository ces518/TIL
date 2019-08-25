# RESTAPI - Security - Spring Security Config 2

#### Spring Security 설정 변경
- HttpSecurity
    - 익명 사용자 요청을 허용
    - form 인증방식 사용
    - GET /api/** 의 모든요청을 허용
    - 나머지 요청들은 인증이 필요하도록 설정
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .anonymous()
            .and()
        .formLogin()
            .and()
        .authorizeRequests()
            .mvcMatchers(HttpMethod.GET, "/api/**").anonymous()
            .anyRequest().authenticated();
}
```

#### 문제점
- Spring Security에서는 PasswordEncoder를 사용하여 동일 여부를 판단하지만
- 현재 AccountService 는 PasswordEncoder를 사용하여 유저를 등록하지 않는다.

#### AccountService 
- AccountService 에서 PasswordEncoder를 사용하여 등록하도록 변경
```java
@Service
public class AccountService implements UserDetailsService {

    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    public Account saveAccount (Account account) {
        account.setPassword(this.passwordEncoder.encode(account.getPassword()));
        return this.accountRepository.save(account);
    }
}
```

#### 테스트 코드
- AccountRepository 를 사용하여 직접 사용자를 등록하지않고
- AccountService 를 사용하여 등록후, 등록된 사용자의 Password와 입력값인 Password를 PasswordEncoder를 사용하여 비교
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("test")
public class AccountServiceTest {

    @Autowired
    AccountService accountService;

    @Autowired
    PasswordEncoder passwordEncoder;

    @Test
    public void loadUserByUsername () {
        // given
        final String username = "juneyoung@email.com";
        final String password = "juneyoung";

        Set roles = new HashSet();
        roles.add(AccountRole.ADMIN);
        roles.add(AccountRole.USER);

        Account account = Account.builder()
                .email(username)
                .password(password)
                .roles(roles)
                .build();

        this.accountService.saveAccount(account);

        UserDetailsService userDetailsService = accountService;
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);


        assertThat(this.passwordEncoder.matches(password, userDetails.getPassword())).isTrue();
    }
}
```

#### ApplicationRunner
- ApplicationRunner를 사용하여 Application 실행시 사용자 하나를 등록하도록 설정
```java
@Bean
public ApplicationRunner applicationRunner () {
    return new ApplicationRunner() {

        @Autowired
        AccountService accountService;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            Account account = Account.builder()
                    .email("pupupee9@gmail.com")
                    .password("june")
                    .roles(Set.of(AccountRole.ADMIN, AccountRole.USER)).build();
            accountService.saveAccount(account);
        }
    };
}
```


- Application 실행시 pupupee9@gmail.com/june 계정이 생성되고, 인증 필요시 해당 사용자로 인증을 진행하면된다.
