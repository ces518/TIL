# Spring Security - Test 2
- Spring Security Test Form Login 기능
- formLogin 기본설정을 사용하고 있지만 password, url 등 formLogin 과 관련된 설정을 커스터마이징이 가능하다.
- 그러한 설정한 내용들이 제대로 동작하는지 테스트가 가능하다. 

#### FormLogin 
- spring-security-test의 formLogin
- june/1234 로 formLogin 을 시도하면 폼 로그인이 성공하는것을 기대하는 테스트 코드 이다.
- 아래의 코드를 살펴보자.
```java
@Test
public void login () throws Exception {
    mockMvc.perform(formLogin().user("june").password("1234"))
        .andExpect(authenticated());
}
```

- 기존 테스트들은 get(), post(), put() 등 httpMethod와 관련된 method를 사용하였지만 spring-security-test 의 formLogin()을 사용한다.
    - user(): 유저명
    - password(): 유저의 패스워드
- authenticated(): 폼로그인이 성공하는것을 기대한다.

위의 테스트코드는 실패한다.

#### 실패한 이유 ?
- spring-security-test 의 formLogin 은 기존에 존재하는 유저정보를 기반으로 테스트를 진행 한다.
- june/1234에 해당하는 유저가 없기때문에 당연히 테스트는 실패한다.

유저가없다면 ? 테스트용 유저를 생성해 주어야한다.

#### 테스트 유저 생성하기
- 먼저 AccountService를 주입받는다.

```java
@Autowired
AccountService accountService;
```
- 그런 다음, username, password를 인자로 받아 테스트용 유저를 생성해주는 method를 정의한다.
```java
private Account createUser(String username, String password) {
    Account account = new Account();
    account.setUsername(username);
    account.setPassword(password);
    account.setRole("USER");
    return accountService.createAccount(account);
}
```

위에서 생성한 테스트용 유저를 생성하는 method를 활용해 테스트 유저를 생성한뒤, 테스트를 진행해보자

변경된 테스트코드는 다음과 같다.
```java
@Test
public void login () throws Exception {
    // given
    final String username = "june";
    final String password = "1234";
    Account createdAccount = createUser(username, password);

    //when & then
    mockMvc.perform(formLogin().user(username).password(password))
        .andExpect(authenticated());
}
```

june/1234에 해당하는 테스트용 유저를 생성해주고, june/1234로 formLogin을 시도하는 테스트를 진행한다.
june/1234에 해당하는 유저가 존재하기때문에 테스트는 성공한다.

그렇다면 이번에는 잘못된 패스워드를 입력하여, 로그인에 실패하는 테스트를 작성해보자

#### 잘못된 패스워드 입력 테스트
- june/1234 에 해당하는 테스트용 유저를 생성한뒤, june/!2345678 로 formLogin 을 시도하면 폼 로그인이 실패하는것을 기대하는 테스트 코드 이다.
- 코드는 이전과 크게 달라진것이 없으며 expect 부분만 달라졌다.
- 전체 테스트코드를 실행해보자.
```java
@Test
public void bad_credentials () throws Exception {
    // given
    final String username = "june";
    final String password = "1234";
    Account createdAccount = createUser(username, password);

    //when & then
    mockMvc.perform(formLogin().user(username).password("!2345678"))
            .andExpect(unauthenticated());
}
```

- unauthenticated()
    - 폼 로그인이 실패하는것을 기대한다.


테스트는 실패한다.

#### 테스트가 실패한 이유 ?
- login() 에서 생성한 유저정보와, bad_credentials() 에서 생성하는 유저정보가 중복되기 때문에 unique 제약조건에 위배되어 유저정보를 생성하지 못한다.
- 따라서 테스트가 실패하는 것이다.
- login() 테스트가 bad_credentials() 테스트에까지 영향을 미치는 것이다.
    - 다른 테스트에 영향을 미치는 테스트는 좋지 못한 테스트이다.


#### @Transactional
- @Transactional 애노테이션은 두가지가 존재한다.
    - javax.transaction.Transactional
    - org.springframework.transaction.annotation.Transactional
- javax, spring 모두 사용이 가능하다.
- spring의 @Transactional 애노테이션도 java 표준을 지키고있으며, spring에서 제공하는 기능이 좀 더 많기때문에 spring의 @Transactional 을 사용하도록 하자.

- 테스트코드에서 데이터베이스에 변경이 일어나는경우 @Transactional 을 사용하여 트랜잭션 처리를 해주어야 한다.
- 테스트 코드에서의 트랜잭션은 기본적으로 해당 테스트가 종료된뒤 롤백된다.
- 다른 테스트에 영향을 주지않는 독립적인 테스트코드가 된다.
- 다른 테스트에 영향을 주는 테스트는 좋지않은 테스트 이다.

- javax.transaction.Transactional
```java
@Inherited
@InterceptorBinding
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Transactional {
    Transactional.TxType value() default Transactional.TxType.REQUIRED;

    @Nonbinding
    Class[] rollbackOn() default {};

    @Nonbinding
    Class[] dontRollbackOn() default {};

    public static enum TxType {
        REQUIRED,
        REQUIRES_NEW,
        MANDATORY,
        SUPPORTS,
        NOT_SUPPORTED,
        NEVER;

        private TxType() {
        }
    }
}
```

- org.springframework.transaction.annotation.Transactional
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    Propagation propagation() default Propagation.REQUIRED;

    Isolation isolation() default Isolation.DEFAULT;

    int timeout() default -1;

    boolean readOnly() default false;

    Class<? extends Throwable>[] rollbackFor() default {};

    String[] rollbackForClassName() default {};

    Class<? extends Throwable>[] noRollbackFor() default {};

    String[] noRollbackForClassName() default {};
}
```


#### @Transactional 을 적용한 테스트 코드
```java
@Test
@Transactional
public void login () throws Exception {
    // given
    final String username = "june";
    final String password = "1234";
    Account createdAccount = createUser(username, password);

    //when & then
    mockMvc.perform(formLogin().user(username).password(password))
        .andExpect(authenticated());
}


@Test
@Transactional
public void bad_credentials () throws Exception {
    // given
    final String username = "june";
    final String password = "1234";
    Account createdAccount = createUser(username, password);

    //when & then
    mockMvc.perform(formLogin().user(username).password("!2345678"))
            .andExpect(unauthenticated());
}
```

@Transactional 을 적용하여 독립적인 테스트가 되었기때문에, 전체 테스트를 실행하여도 테스트가 모두 성공하는것을 확인할 수 있다.
