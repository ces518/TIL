# HTTP 
- HTTP 란 ?
   - Hyper Text Transfer Protocol 
   - 인터넷상에서 데이터를 주고 받기위한 서버/클라이언트 모델을 따르는 프로토콜 애플리케이션 레벨의 프로토콜로 TCP/IP 위에서 동작한다.


- 작동방식
   - 클라이언트에서 request를 보내면 서버는 reponse를 한다.

- Connectionless
   - HTTP는 Connectionless 방식으로 작동한다. 
   - 서버에 연결을 하고 요청해서 응답을 받으면 연결을 끊어버린다.

- 장점 
- 불특정 다수를 대상으로하는 서비스에 적합하다.

- 단점
- 연결을 끊어버리기 때문에, 클라이언트의 이전 상태를 알 수가없다. 이런 특징을 stateless 라고한다.


- URI
   - 클라이언트 프로그램은 URI를 이용하여 자원의 위치를 찾는다.
   - URI는 HTTP와 독립된 다른체계이다.
   - HTTP는 전송프로토콜, URI는 자원의 위치를 알려주기위한 프로토콜이다.


-  Method
   - 메서드는 요청의 종류를 서버에게 알려주기위해서 사용한다.
   - GET : 정보를 요청하기 위해서 사용한다.
   - POST : 정보를 밀어넣기 위해서 사용한다.
   - PUT : 정보를 업데이트하기 위해서 사용한다.
   - DELETE : 정보를 삭제하기 위해서 사용한다.
   - HEAD : 헤더 정보만 요청한다. 해당자원이 존재하는지, 서버에 문제가없는지 확인용
   - OPTIONS : 웹서버가 지원하는 메서드의 종류를 요청한다.
   - TRACE : 클라이언트의 요청을 그대로 반환한다. echo서비스로 서버 상태를 확인하기위한 목적으로 주로사용한다.


- 요청 데이터
   -요청 데이터는 header 와 body로 구성된다.
   - http 헤더는 라인피드와 캐리지 리턴을 함께사용한다 (\r\n) http 헤더파싱시 주의.

- Header 정보 
```
===============================================
1 GET /cgi-bin/http_trace.pl HTTP/1.1\r\n
2 ACCEPT_ENCODING: gzip,deflate,sdch\r\n
3 CONNECTION: keep-alive\r\n
4 ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n
5 ACCEPT_CHARSET: windows-949,utf-8;q=0.7,*;q=0.3\r\n
6 USER_AGENT: Mozilla/5.0 (X11; Linux i686) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.24\r\n 
7 ACCEPT_LANGUAGE: ko-KR,ko;q=0.8,en-US;q=0.6,en;q=0.4\rn
8 HOST: www.joinc.co.kr\r\n
9 \r\n
===============================================
```
1. 필수 요소로 요청의 젤 처음에 와야한다. 3개의필드로 이뤄져있다.
ㄴ 1. 요청메서드.
   2. 요청 URI
   3. HTTP 프로토콜 버전


- 응답 헤더 포멧
```
===============================================
1 HTTP/1.1 200 OK\r\n
2 Date: Fri, 08 Jul 2011 00:59:41 GMT\r\n
3 Server: Apache/2.2.4 (Unix) PHP/5.2.0\r\n
4 X-Powered-By: PHP/5.2.0\r\n
5 Expires: Mon, 26 Jul 1997 05:00:00 GMT\r\n
6 Last-Modified: Fri, 08 Jul 2011 00:59:41 GMT\r\n
7 Cache-Control: no-store, no-cache, must-revalidate\r\n
8 Content-Length: 102\r\n
9 Keep-Alive: timeout=15, max=100\r\n
10 Connection: Keep-Alive\r\n
11 Content-Type: text/html\r\n
\r\n
===============================================
```
1. 프로토콜과 응답코드, 반드시에 첫줄에와야한다 3개의 필드로 구성
ㄴ 1. HTTP/1.1 응답 프로토콜과 버전
   2. 200 : 응답 코드
   3. OK : 응답 메세지 

2. 날짜
3. 서버프로그램 및 스크립트 정보 
4. 응답헤더에 다양한 정보 추가가능. 
5. 컨텐츠 마지막 수정일
6. 캐시 제어방식
7. 컨텐츠길이
8. Kepp ALive 기능설정

