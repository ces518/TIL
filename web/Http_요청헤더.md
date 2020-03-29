#### HTTP 요청 헤더
`Host`
- 서버의 도메인 네임이 나타나는 부분 (포트를 포함)
- Host 헤더는 반드시 하나가 존재해야 함
- Host: www.naver.com

`User-Agent`
- 현재 사용자가 어떤 클라이언트를 이용해 요청을 보냈는지 나오게 된다.
    - 운영체제, 브라우저 등
- Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Whale/1.4.64.5 Safari/537.36

`Accept`
- Accept 헤더는 요청을 보낼 때 서버에 이러한 타입 (MIME)의 데이터를 보내달라고 명시할때 사용
- 콤마로 여러타입을 동시에 적어줄 수 있고, 와일드카드를 사용할 수도 있다.
- Accept: text/html
- Accept: image/png, image/gif
- Accept: text/*

`Authorization`
- 인증 토큰을 서버로 보낼때 사용하는 헤더이다.
- JWT, Bearer emd..
- API 요청시 같이 사용한다.

`Origin`
- 요청을 보낼때 어느 주소에서 시작되었는지를 나타낸다. 보낸주소와 받는 주소가 다르면 CORS문제가 발생하기도 한다.

`Referer`
- 이전 페이지의 주소정보가 담겨 있다.
- 어떤 페이지서 이동했는지 알수 있기 때문에 구글 애널리틱스 같은데에서 많이 이용된다.
