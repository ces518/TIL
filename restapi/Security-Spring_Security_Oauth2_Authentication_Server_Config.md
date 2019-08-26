# REST API - Security - Spring Security - OAuth2 Authorization Server Config
#### OAuth2 인증 방법
- Spring OAuth2 에서 지원하는 6가지 방법중 2가지를 사용하여 인증 서비스를 지원 한다.
- Password와 Refresh Token 방식

#### Password
- Spring OAuth2가 지원하는 방법중 하나
- Twitter 로그인시 ID와 PASSWORD를 직접 입력하여 여러 홉을 거칠 필요없이 바로 로그인하는것처럼 PASSWORD를 직접 입력하는 방식이기때문에 PASSWORD라는 이름이 붙음.
- 최초 발급시 사용하는 방법
- 홉이 1번만 일어난다.
    - 요청과 응답이 1쌍이다.
- 보통의 방법은 홉이 많음
    - Token을 발급 받을수 있는 Token을 발급하고 Token을 사용해서 인증하도록 Redirection하는등 .. 홉이 많음
- 이 인증 방식은 FirstClass 즉 인증을 제공하는 서비스들이 만든 앱이 사용하는 방식이다.
- 서드파티 앱에서 사용하도록 허용해주어선 안된다.
    - 사용자의 정보 (ID, Password)를 직접 입력하기 때문에 보안적 문제..

- 필요정보
    - GrantType, username, password 
        - 파라메터 방식으로 전달
    - ClientId, ClientSecret 

- 장점
    - 홉이 적다.
    - access_token을 바로 발급할 수 있다.

#### OAuth2 인증서버 테스트
- 인증 토큰을 받는 테스트
- POST /oauth/token 으로 요청을 보내면 access_token 이 발급 되기를 기대한다.
- 요청 HEADER에 httpBasic() 을 사용하여 basicOauth HEADER를 만들어 요청에 같이 보낸다.
- 요청 Parameter로 grant_type, username, password 을 전달한다.
```java
public class OAuthServerConfigTest extends BaseControllerTest {

    @Autowired
    AccountService accountService;

    @Test
    @TestDescription("인증 토큰을 발급 받는 테스트")
    public void getAuthToken () throws Exception {

        final String USER_NAME = "puppee9@gmail.com";
        final String PASSWORD = "june";
        // httpBasic 메서드를 사용하여 basicOauth 헤더를 만듬
        final String CLIENT_ID = "myApp";
        final String CLIENT_SECRET = "pass";

        // given
        Set roles = new HashSet();
        roles.add(AccountRole.ADMIN);
        roles.add(AccountRole.USER);
        Account account = Account.builder()
                .email(USER_NAME)
                .password(PASSWORD)
                .roles(roles)
                .build();
        accountService.saveAccount(account);

        this.mockMvc.perform(post("/oauth/token")
                    .with(httpBasic(CLIENT_ID, CLIENT_SECRET)) // httpBasic 사용시 test dependency 필요
                    .param("username", USER_NAME)
                    .param("password", PASSWORD)
                    .param("grant_type", "password")
                )
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.access_token").exists());
    }

}
```

#### SpringSecurityTest
- httpBasic() 메서드를 사용하려면 SpringSecurityTest가 의존성에 존재해야한다.
- 다음과 같이 pom.xml에 spring-security-test 의존성을 추가한다.
```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <version>${spring-security.version}</version>
    <scope>test</scope>
</dependency>
```

#### OAuthServerConfig
- @EnableAuthorizationServer 를 사용하여 인증 서버 설정 활성화
- security 관련 설정
    - clientSecret을 사용할때 passwordEncoder를 사용하기때문에 bean으로 주입받아 사용한다.
- client 관련 설정
    - inMemory로 관리한다.
    - client-id: myApp
    - client-secret: pass
        - passwordEncoder를 사용하여 확인하기 때문에 encoding을 해주어야 한다.
    - access-token 유효시간을 10분으로 설정한다.
    - refresh-token 유효시간을 1시간으로 설정한다.
- endpoint 관련 설정
    - authenticationManager
        - 사용자 정보를 알고있는 authenticationManager 를 등록한다.
        - 이전의 Security 설정에서 authenticationManager 를 빈으로 등록을 해 주었기 때문에 의존성 주입을 받아 사용한다.
    - userDetailsService
    - tokenStore
        - token을 저장소
        - 마찬가지로 이전의 security 설정에서 빈으로 등록을 해 주었기 때문에 의존성 주입을 받아 사용한다.
```java
@Configuration
@EnableAuthorizationServer
public class OAuthServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    PasswordEncoder passwordEncoder;

    @Autowired
    AuthenticationManager authenticationManager;

    @Autowired
    AccountService accountService;

    @Autowired
    TokenStore tokenStore;

    // 패스워드 인코더를 사용하여 클라이언트 시크릿을 확인한다.
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.passwordEncoder(passwordEncoder);
    }

    /**
     * inMemory 로 관리
     * clientId는 myApp
     * grant-type은 password와 refresh_token 방식 지원
     * scopes 은 정의하기 나름 read, write
     * client-secret은 passwordEncoder로 encoding
     * accessToken 유효시간 10분설정
     * refreshToken 유효시간 1시간
     * @param clients
     * @throws Exception
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("myApp")
                .authorizedGrantTypes("password", "refresh_token")
                .scopes("read", "write")
                .secret(this.passwordEncoder.encode("pass"))
                .accessTokenValiditySeconds(10 * 60)
                .refreshTokenValiditySeconds(6 * 10 * 60)
        ;
    }

    /**
     * authenticationManager : 사용자정보를 가지고있음
     *  - 유저정보를 확인해야 토큰을 발급받을수 있음
     * accountService : userDetailsService
     * tokenStore : 토큰을 저장할 tokenStore
     * @param endpoints
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(authenticationManager)
            .userDetailsService(accountService)
            .tokenStore(tokenStore);
    }
}
```

#### 테스트 결과
- POST /oauth/token
- {username=[puppee9@gmail.com], password=[june], grant_type=[password]}
- [Authorization:"Basic bXlBcHA6cGFzcw=="]
- 요청을 보냈을때 access_token과 refresh_token 이 발급되고, 앞선 설정에서 적용한 토큰 유효시간과 scope이 정상적으로 적용된것을 확인할 수 있다.
```java
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /oauth/token
       Parameters = {username=[puppee9@gmail.com], password=[june], grant_type=[password]}
          Headers = [Authorization:"Basic bXlBcHA6cGFzcw=="]
             Body = null
    Session Attrs = {}

Handler:
             Type = org.springframework.security.oauth2.provider.endpoint.TokenEndpoint
           Method = public org.springframework.http.ResponseEntity<org.springframework.security.oauth2.common.OAuth2AccessToken> org.springframework.security.oauth2.provider.endpoint.TokenEndpoint.postAccessToken(java.security.Principal,java.util.Map<java.lang.String, java.lang.String>) throws org.springframework.web.HttpRequestMethodNotSupportedException

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Pragma:"no-cache", Cache-Control:"no-store", Content-Type:"application/json;charset=UTF-8", X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", X-Frame-Options:"DENY"]
     Content type = application/json;charset=UTF-8
             Body = {"access_token":"2e5d1d80-3522-4bbe-94d4-bac0b1ccfcc2","token_type":"bearer","refresh_token":"4cff2846-c687-41f0-a3dc-e312c7a56302","expires_in":599,"scope":"read write"}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
