#### HTTP 공통 헤더
- 요청과 응답에 모두 사용되는 헤더이다.
- 이중 Content로 시작하는 헤더는 엔티티 헤더라고 불린다.

`Date`
- HTTP 메시지가 만들어진 시각
- Date: Sun, 29 Mar 2020 15:24:48 GMT

`Connection`
- HTTP/2 를 사용하지 않는다면 보통 HTTP/1.1 을 사용한다.
- 기본적으로 keep-alive 로 되어있지만 아무런 의미가 없다.
- HTTP2에서는 사라져 버린 헤더

`Cache-Control`
- 아무것도 캐싱하지 않을떄
    - Cache-Control: no-store

> no-cache이지만 캐시를 사용하지 말라는 의미가 아닌 모든 캐시를 쓰기전 서버에 물어보라는 의미이다.

- 만료된 캐시만 서버에 확인을 받도록 할때
    - Cache-Control: must-revalidate
- 공유 캐시에 저장해도 될때
    - Cache-Control: public
- 브라우저와 같은 특정 사용자 환경에만 저장할 때
    - Cache-Control: private
- 캐시의 유효시간을 줄때
    - 단위는 초 단위이다.
    - Cache-Control: public, max-age=3600

> 위의 옵션들은 모두 혼합해서 사용이 가능하다.

`Content-Length`
- 요청과 응답 메시지의 본문 크기를 바이트 단위로 표시해준다.
- 메시지 크기에 따라 자동으로 생성된다.
- Content-Length: 40

`Content-Type`
- 컨텐츠 타입(MIME)과 문자열 인코딩을 명시할 수 있다.
- Accept헤더, Accept-Charset헤더에 대응된다.
- Content-Type: text/html; charset=utf-8

`Content-Language`
- 사용자의 언어를 의미한다.
- 요청이나 응답이 무슨 언어인지와는 관련 없다.

`Content-Encoding`
- 컨텐츠 압축 방식
- 응답 컨텐츠를 br, gzip등의 알고리즘으로 압축해서 보내면, 브라우저가 알아서 해제해서 사용한다.
- 컨텐츠 용량이 줄어들기 때문에 되도록 압축을 권장함
- Content-Encoding: gzip, deflate
