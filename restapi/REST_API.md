# REST API

### API
- Application Programming Interface

### REST
- Represntational State Transfer
- 인터넷 상의 상호 운용성을 제공하는 방법중 하나
	- 웹을 깨뜨리지않으면서 HTTP를 진화시키는 방법
- 시스템 제 각각의 독립적인 진화를 보장하기 위한방법
- REST API: REST 아키텍처 스타일을 따르는 API

#### REST API 스타일
- Client-Server
- Stateless
- Cache
- Uniform Interface
- Layered System
- Code-On-Demand (optional)

#### Uniform-interface
- Identification of resources
- manipulation of resources through represneations
- self-descriptive messages
	- 메시지 스스로 메시지에 대한 설명이 되어야한다.
	- 서버가 변해서 메시지가 변해도 클라이언트는 해석이 가능해야한다.
	- 확장 가능한 커뮤니케이션
- hypermiss as the engine of application state (HATEOAS)
	- 하이퍼미디어 를 통해 애플리케이션 상태 변화가 가능해야한다.
	- 링크 정보를 동적으로 바꿀수 있어야한다. (Versioning없이)

#### 오늘날 REST API 
- 오늘날 대부분의 REST API 들은 RESTAPI가 아니다.
- Uniform-interface 중 self-description, HATEOAS가 가장 지켜지지않는다.
- 대부분은 HTTP API, WEB API 이다.

#### Self-descriptive-message 해결방법
- 1. 미디어타입을 정의하고 IANA에 등록하고 , 그 미디어타입을 리소스의 Content-type 으로 사용한다.
- 2. profile 링크 헤더를 추가한다.
	- 대부분의 브라우저들이 아직 스펙 지원을 잘 하지않는다.
	- 대안으로 HAL의 링크 데이터에 profile링크 추가

#### HATEOAS 해결방법
- 1. 데이터에 링크를 제공
	- 데이터 정의 방법 ? HAL
- 2. 링크 헤더나 Location을 제공한다.

#### HAL
- Hypertext Application Language
- https://en.wikipedia.org/wiki/Hypertext_Application_Language

#### 구현할 방법
- HAL 스펙을 활용해서 링크를 제공하는 방법으로 해결
- 헤더에 추가하지않고, 응답본문에 추가
	- 대부분의 클라이언트들이 해석하지못한다.

- RESTAPI의 좋은 예 
	- Github
