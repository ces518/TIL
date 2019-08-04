# REST API - OAuth 2.0

#### OAuth
- 3가지의 주체가 등장한다.
	- 1. 나의 서비스
	- 2. 사용자
	- 3. 나의 서비스와 연동하려는 다른 서비스들

- 연동하는 가장 간단한 방법
	- 사용자 입장에서의 다른 서비스 (Google, Facebook) 의 계정정보를 우리 서비스에 저장
	- 다른 서비스와 연동할때 모든 기능을 사용할수 있기때문에 매우 간단하면서 강력함
	- 하지만 매우 위험한방법이다.
	- 신뢰할수 있는 서비스 일까 ?..

- 이런 문제를 해결하기 위해 나온것이 OAuth
	- 사용자의 요청에 의해서 연동하고싶은 다른 서비스의 ID, Password 가 아닌 AccessToken 을 발급받아 해당 Token을 활용한다.
	- 모든 기능을 활용하는것이 아닌 부분적인 기능을 활용할수 있다.

#### OAuth의 역할
- 3가지 주체의 용어 정의
	- 1. 나의 서비스
		- Client
	- 2. 사용자
		- Resource Owner
	- 3. 나의 서비스와 연동하려는 다른 서비스들
		- Resource Server: 데이터를 가지고 있는 서버
		- Authorization Server: 인증 서버

- 위의 3가지 주체가 OAuth의 핵심이다.

#### OAuth 등록
- 클라이언트가 Resource-Server를 사용하기 위해서는 Resource Server로 부터 사전의 승인을 받아야하는데
- 이를 등록이라고 한다.
- 등록하는 방법은 서비스마다 다르지만 공통적인 부분이 존재한다.

- 공통적인 부분
```
ClientID: 1 (Application 식별자 [노출되어도 됨])
ClientSecret: 2 (비밀번호 [노출이 되어선 안된다.])
Authorized Redirect Urls: https://client/callback: 요청을 허용하는 Url
```

#### Resource-Owner의 승인
- 등록을 하게되면 Client와 Resource-Server가 아래의 공통적인 정보를 알고 있게 된다.
- Client는 Authorized Redirect Urls 에 해당하는 페이지를 구현하고 있어야한다.
```
ClientID: 1 (Application 식별자 [노출되어도 됨])
ClientSecret: 2 (비밀번호 [노출이 되어선 안된다.])
Authorized Redirect Urls: https://client/callback: 요청을 허용하는 Url
```

- A, B, C, D 의 기능중 B와 C의 기능만 필요하다고 할경우, 모든 기능에 대해 인증을 받는것이 아닌 최소한의 인증을 받는것이 양쪽 모두에게 효율적이다.
- Resource-Owner가 우리 서비스에 존재하는 게시글을 Facebook 또는 Twitter에 공유하는것을 Client에 요청을 할경우 Facebook, Twitter에 로그인 (사용자의 동의를 구하는것) 해야한다는 응답을 보낼것이다.
- 사용자가 동의를 할 경우에만 다음 단계로 넘어갈 것이다.
- Resource-Owner가 로그인에 성공했다면, Resource-Server는 ClientID에 해당하는 클라이언트가 존재하는지 찾는다.
- 해당 요청의 RedirectUrl와 동일한지 확인한다. 다를경우 처리를 끝내버린다.
- Url이 동일하다면, 어떠한 기능에 대한 권한을 요청하고있으며 이를 승인하는것인지 Resource-Owner에게 묻는다.
- 이를 승인했다면 Client는 Resource-Server로부터 해당 Resource에 대한 권한을 얻게 된다.

#### Resource-Server의 승인
- Resource-Server는 Access-Token을 바로 발급하지 않고, 전 인증 단계가 존재한다.
- 임시비밀번호를 사용해서 인증절차를 걸치는데 이를 Authorization Code 라고 한다.
- Resource-Server는 이를 Resource-Owner에게 전송한다.
	- Resouce-Server가 Resource-Owner에게 임시비밀번호와 함께 Location 헤더에 의해 Redirect Url 보낸다. 
	- Resource-Owner는 해당 페이지로 이동함으로써 Client는 Authorization Code를 알게 된다.
- 해당 코드를 이용하여 Client는 Resource-Server로 요청을 보내게된다.
- Resource-Server를 Authorization Code 와 ClientId, ClientSecret, Url 모두가 일치할 경우 AccesToken을 발행한다.

#### Access-Token 발행
- Client 는 Resource-Owner를 통해서 Authorization Code를 알게 되고 이를 Client정보와 함께 Resource-Server로 전송하게 된다.
- 그 다음 단계가 바로 Access-Token을 발급한다.
- Resource-Server는 Authorization Code를 지워버리게된다.
- 그런 다음 Access-Token을 발급하여, Client에게 응답한다.
- Client는 응답받은 Access-Token을 저장해둔다.
- Resource-Server는 Access-Token을 보고 1이라는 유저의 B와 C에 대한 기능에 대한 인증이 된 token이라 인식하고, 해당 Resource를 제공하게 된다.

#### API
- Application Programming Interace
- Client가 Resource-Server로 부터 Resource를 제공받기 위한 접점이라고 생각하자.
- API는 Resource-Server마다 제 각각이다.
- API 요청 방식은 2가지로 나뉜다.
	- 1. access_token을 URI Parameter로 전송하는 방법
	- 2. Authorization: Bearer <access_token> HEADER로 인증하는 방법
	- 후자의 경우가 더 안전하며 더 선호되는 방법이다.
	- access_token을 전송하는 표준화된 방식

#### Refresh_token
- Access_token는 수명이 존재한다.
- 해당 수명이 존재하면 더이상 API를 사용했을때 데이터를 주지 않는다.
- 이런 Access_token을 갱신해야하는데 그럴때마다 사용자에게 요청하는것은 제한된다.
- 이를 손 쉽게 하는것을 Refresh_token이라고 한다.
- REC 6749
	- https://tools.ietf.org/html/rfc6749
- client_id, client_secret, refresh_token 정보를 Resource-Server로 보낼경우 access_token이 재발급 된다.

- OAuth를 사용하는 궁극의 목적은 API 를 제어하는것이다.
