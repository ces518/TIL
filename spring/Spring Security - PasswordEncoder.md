# Spring Security - PasswordEncoder
- Domain 로직의 {noop} + password 를 passwordEncoder에게 역할을 위임 하도록 한다.

#### PasswordEncoder 를 빈으로 등록하기
- SpringSecurity 5 이전 버전에서 사용하던 NoOpPasswordEncoder 를 빈으로 등록한다.
- Deprecated 되어 권장하지 않는 방법이다.
```java
@Bean
public PasswordEncoder passwordEncoder () {
	return NoOpPasswordEncoder.getInstance();
}
```

#### PasswordEncoder 적용하기
- PasswordEncoder를 주입받아 Account 로 넘겨준뒤 인코딩 작업을 진행한다.

* AccountService
```java
@Service
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    private final AccountRepository accountRepository;
    private final PasswordEncoder passwordEncoder;

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

    public Account createAccount(Account account) {
        account.encodePassword(passwordEncoder);
        return accountRepository.save(account);
    }
}
```

- 기존의 {noop} + passwordEncoder 로 인코딩 작업을 해주던 로직을 passwordEncoder를 사용하는 방식으로 변경한다.

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

    public void encodePassword(PasswordEncoder passwordEncoder) {
        this.password = passwordEncoder.encode(this.password);
    }
}
```

#### 테스트
- passwordEncoder를 사용하는 방식을 사용하면 어떤식으로 동작할지 한번 테스트 해보자.
- 이전과 마찬가지로 http://localhost:8080/account/USER/user/1234 로 요청을 보내 유저를 생성해보자.
- 다음과 같이 유저가 생성되고 응답이 오는것을 확인할 수 있다.
	- Security 5에서 사용하던 방식인 {noop} prefix 문자열이 사라진것이 확인된다.
- 생성된 유저정보로 인증을 하면, /dashboard 도 접근이 가능하다.
```javascript
{"id":1,"username":"user","password":"1234","role":"USER"}
```

#### PasswordFormat이 생긴 이유 ?
- 이전에서는 noop passwordEncoder 였지만
- 기본 전략이 {noop} 에서 bcrypt 방식으로 바뀌었다.
- 이로서 문제가 발생하였음.
	- 이전의 버전을 사용하던 사람들은 평문으로 사용했을 수도 있다.
	- 다른 암호화 방식을 사용하고 싶은 사람도 있다.
- 다양한 패스워드 방식을 제공하기위해 format이 변경되었다.


#### SpringSecurity 5.x 기본 PasswordEncoder
- sha256, scrypt 등 id에 해당하는 알고리즘 명을 주고, 실제 패스워드 인코더 객체를 설정해주면, 알고리즘에 해당하는 패스워드 인코드를 사용한다.
- SpringSecurity 5 에서 기본 패스워드 인코더로 사용하는 bcrypt 방식을 암호화를 사용하는 PasswordEncoder를 사용해보자

* Bcrypt PasswordEncoder를 Bean으로 등록
```java
@Bean
public PasswordEncoder passwordEncoder () {
	return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```


* http://localhost:8080/account/USER/user/1234 로 유저생성하기
- 다음과 같이 bcrypt 인코딩이 된걸 확인할 수 있다.
```java
{"id":1,"username":"user","password":"{bcrypt}$2a$10$5/xnEDQjK2lqKjgSRu9YEuq8.XEuMFetQl5R6EavMXAsWV1CW3aji","role":"USER"}
```
