# REST API - Security - Account Service
#### AccountServiceTest
- 기존에 만들어둔 BaseControllerTest 클래스는 ServiceLayer에선 사용하지않는 MockMvc를 포함하고 있기때문에 상속하지 않는다.
- UserDetailsService를 구현하는 AccountService 테스트

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("test")
public class AccountServiceTest {

    @Autowired
    AccountService accountService;

    @Autowired
    AccountRepository accountRepository;

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

        this.accountRepository.save(account);

        UserDetailsService userDetailsService = accountService;
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);


        assertThat(userDetails.getPassword()).isEqualTo(userDetails.getPassword());
    }

    /* 타입밖에 확인하지 못한다. */
    @Test(expected = UsernameNotFoundException.class)
    public void findByUsernameFail () {
        final String username = "random@gmail.com";
        accountService.loadUserByUsername(username);
    }
}
```

- AccountService
    - loadUserByUsername()
        - username(email) 에 해당하는 사용자를 찾는다.
            - 없다면 UsernameNotFoundException
            - 존재한다면 UserDetail 구현체인 User 객체를 반환한다.
    - authorities()
        - username에 해당하는 사용자를 찾아 해당 사용자의 권한을 SimpleGrantedAuthority로 변환한다.   
```java
@Service
public class AccountService implements UserDetailsService {

    @Autowired
    private AccountRepository accountRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account account = accountRepository.findByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException(username));
        return new User(account.getEmail(), account.getPassword(), authorities(account.getRoles()));
    }

    private Collection<? extends GrantedAuthority> authorities(Set<AccountRole> roles) {
        return roles.stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role.name()))
                .collect(Collectors.toSet());
    }
}
```

- AccountRepository
    - findByEmail()
        - email에 해당하는 사용자 정보를 찾는다. Optional
```java
public interface AccountRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByEmail(String username);
}
```
