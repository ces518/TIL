#### HTTP 응답 헤더
`Access-Control-Allow-Origin`
- 요청을 보내는 프론트주소와 요청을 받는 백엔드주소가 다르면 CORS 에러가 발생한다.
- 이 때 서버에서 응답 메시지 Access-Control-Allow-Origin 헤더에 프론트 주소를 적어주어야 에러가 나지 않는다.
- 프로토콜, 서브도메인, 도메인, 포트 중 하나만 달라도 CORS에러가 발생한다.

`Allow`
- Allow 헤더는 Access-Control-Allow-Methods와 비슷하지만 CORS요청 외에도 적용된다는 차이점이 있다.
- GET www.naver.com 은 허용하지만, POST www.naver.com은 허용하지 않는 경우
- 405 Method Not Allowed 에러와 함께 Allow 헤더를 같이 보내면 된다.
- Allow: GET 

`Content-Disposition`
- 응답 본문을 브라우저가 어떻게 표시해야 할 지 알려주는 헤더이다.
- inline일 경우 웹페이지 화면에 표시되고
- attachment인 경우 다운로드 된다.
- Content-Disposition: inline
- Content-Disposition: attachment; filename='filename.csv'

`Location`
- 300번대 응답 혹은 201 Created 응답일 때 어느 페이지로 이동할지를 알려주는 헤더
- HTTP/1.1 302 Found
- Location: /

`Content-Security-Policy`
- 다른 외부 파일을 불러들이는 경우, 차단할 소스와 불러올 소스를 명시할 수 있다.
- self로 지정하면, 자신의 도메인 파일들만 불러올 수 있다.
- default-src는 모든 외부소스에 적용되는 것이고, 각각 따로 지정도 가능해진다.
- Content-Security-Policy: default-src 'self'
- Content-Security-Policy: default-src https:
- Content-Security-Policy: default-src 'none'
