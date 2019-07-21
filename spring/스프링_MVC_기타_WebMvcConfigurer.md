# Spring MVC - WebMvcConfigurer
- 앞서 살펴본 설정 이외에 설정들 ...

- CORS 설정
    - Cross Origin 요청 처리설정
    - 같은 도메인에서 온 요청이 아니더라도 이를 허용하고싶을때 설정

- ReturnValueHandler
    - 스프링 MVC가 제공하는 기본 리턴값 이외에 커스텀한 리턴값을 처리할때 설정한다.

- ArgumentResolver 설정
    - 핸들러에서 사용하는 아규먼트, 스프링에서 지원하지않는 커스텀한 아규먼트들을 추가해서 사용할 수 있따.

- ViewController 설정
    - 핸들러를 작성할필요 없이 심플하게 View로 매핑하고싶을때 사용한다.

- 비동기
    - 비동기 요청 처리시에 사용할 타임아웃이나 TaskExecutor를 설정
- ViewResolver
    - 핸들러에서 리턴하는 뷰 이름에 해당하는 문자열을 View인스턴스로 바꿔줄 뷰 리졸버 설정
- ContentNegotiation
    - 요청 본문 또는 응답본문을 어떤 타입으로 보내야하는지 결정하는 전략 설정

