# REST API - Spring - HATEOAS
- Spring Project 중 하나, REST 한 API를 만들때 representation을 제공하기 편하게 하는 라이브러리

#### HATEOAS
- https://en.wikipedia.org/wiki/HATEOAS
- Hypermedia를 활용하여 ApplicationServer와 정보를 동적으로 주고 받는 방법

- ApplicationServer는 다음과 같이 Client와 정보를 주고 받을때 Hypermedia를 활용하여 리소스의 상태 혹은 상호작용에 따른 링크 정보를 제공해야한다.
- Client 는 URI 가 바뀌더라도 relation 만 보고 소통을 문제 없이 가능해야한다.
```xml
HTTP/1.1 200 OK
Content-Type: application/xml
Content-Length: ...

<?xml version="1.0"?>
<account>
    <account_number>12345</account_number>
    <balance currency="usd">100.00</balance>
    <link rel="deposit" href="/accounts/12345/deposit" />
    <link rel="withdraw" href="/accounts/12345/withdraw" /> 
    <link rel="transfer" href="/accounts/12345/transfer" />
    <link rel="close" href="/accounts/12345/close" />
</account>
```
- 가장 중요한 기능은 Link를 만드는 기능과 Resource 를 만드는 기능이다.

#### HATEOAS에서 Resource 란
    - 응답본문 (데이터) + 링크를 의미한다.

#### Link
- HREF
- REL
    - self: 자신에 대한 URI
    - profile: 응답 본문에 대한 문서의 URI
    - ...
    
#### 이벤트를 생성했을때 어떤 링크를 제공해야하는가 ?
    - 1. SELF
    - 2. PROFILE
    - 3. 이벤트를 수정할수 있는 URI
        - update-event
    - 4. 이벤트를 조회할수 있는 URI
        - query-event
