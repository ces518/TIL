# HTTP Bearer

## Bearer Authentication 이란 ?
- API 에 접속하기 위해 access token 을 이용해서 인증을 해야한다.
- 이 때 사용하는 인증 방법이 Bearer Authentication 이다.
- OAuth 를 위해 고안된 방법이며 RFC 6750 에 명세가 되어있다.
- access token 을 서버로 전송할 때는 반드시 Https 를 사용해야 한다.
- Bearer Authentication 외에도 Form-Encoded Body Parameter 방식과 URI Query Parameter 방식이 존재한다.
- Header 를 사용할 수 없는 경우 위 두가지 방법을 사용하는데, 보안적인 문제로 권장되지 않는다.



# 참고
- https://seomal.org/?i=WEB3-BEARER-AUTHENTICATION
- https://tools.ietf.org/html/rfc6750