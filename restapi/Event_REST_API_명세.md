# REST API - Event Rest API
- 이벤트 등록, 조회 및 수정 API

#### GET /api/events
- 이벤트 목록 조회 REST API (비로그인)
	- 응답에 보여주어야 할 데이터
	- 이벤트목록
	- 링크
		- self
		- profile: 이벤트 목록조회 API 문서로 링크
		- get-an-event: 이벤트를 하나 조회하는 API 링크
		- next: 다음 페이지 (optional)
		- prev: 이전 페이지 (optional)
	- 문서
		- Spring REST Docs 로 생성

- 이벤트 목록 조회 REST API (로그인)
	- 응답에 보여주어야 할 데이터 
	- 이벤트 목록
	- 링크
		- self
		- profile: 이벤트 목록 조회 API 문서로 링크
		- get-an-event: 이벤트 하나를 조회하는 API 링크
		- create-new-event: 이벤트를 생성할 수 이있는 API 링크
		- next: 다음페이지 (optional)
		- prev: 이전 페이지 (optional)

- 로그인 상태 구분 방법 ?
	- Access Token이 존재하는 경우

#### POST /api/events
- 이벤트 생성

#### GET /api/events/{id}
- 이벤트 하나 조회

#### PUT /api/events/{id}
- 이벤트 수정
