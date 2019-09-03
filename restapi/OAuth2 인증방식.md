# OAuth2 인증방식

#### 1. 권한 코드 방식 (Authorization Code)
- 일반적인 웹사이트에서 가장 많이 사용하는 인증방식
- 엑세스 토큰 발급을 위한 테스트가 다른 방법에 비해 복잡하다.
- 인증을 받기위한 code를 발급받고, 해당 code를 이용해 인증을 받는방식

#### 2. 암묵적인 동의 방식 (Implipic Grant)
- 보통 Client Side 에서 OAuth2 인증을 할떄 사용하는 방식이다.
- redirect_uri 를 사용하여 토큰을 발급하고 # (해쉬태그) 뒤로 access_token을 넘겨주기 때문에 서버에서는 access_token을 받지 못한다.
- 리프레시 토큰을 제공하지 않는다.
- 보안상 프론트에서만 access_token을 사용하도록 권장하는 형태의 방식이다.

#### 3. 자원 소유자 비밀번호 (Resource Owner Password Credentials Flow)
- 자원 소유자의 아이디와 패스워드로 access_token을 발급받는 방식.

#### 4. 클라이언트 인증 (Client Credentials Flow)
- 클라이언트가 직접 자신의 정보를 통해 access_token을 발급받는 방식
- 해당 서비스가 직접 인증을 제공하는 경우에만 사용할 것
- 서드파티 앱에서 절대 허용해서는 안된다.
    - 보안 문제

#### 5. 리프래시 토큰 (Refresh Token) 을 통한 방식
- Refresh Token을 활용하여 access_token을 재발급 받는 방식
- 기존에 저장해둔 Refresh Token이 존재할때 access_token을 재발급 받을 필요가 있을경우 사용
