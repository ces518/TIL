# Spring Method Argument, Return Type
- 기본적으로 제공하는 타입들이 다양하며, 원한다면 커스텀한 타입을 지원하도록 설정이 가능하다.

#### HandlerMethodArgument
    - 주로 요청 그 자체 또는 요청에 들어 있는 정보를 받아오는 사용한다.

- org.springframework.web.context.requeset.WebRequest
- org.springframework.web.context.request.NatvieWebRequest
    - Spring이 제공해주는 API
    - ServletAPI를 Wrapping한 형태
    - 요청과 관련된 다양한 정보를 가져올 수 있다.
        - Header 정보
        - Locale 정보
        - Parameter 정보
        - Principal
        - ContextPath
        - SessionId 
        - .. 등
    
- javax.servlet.http.HttpServletRequest, javax.servlet.http.HttpServletResponse
    - ServletAPI
    - Low-Level로 직접 코딩을 하고싶다면 사용할 수 있다.

- java.io.InputStream, java.io.OutputStream
    - ServletAPI를 직접 사용하지 않고, 요청본문을 읽어오고 싶을때 사용할 수 있다.
    - InputStream 자체가 Requestbody이다.
    - OutputStream 으로 응답한다면 ResponseBody가 된다.

- java.io.Reader, java.io.Writer
    - 위 두 Stream들을 추상화 한 것

- javax.servlet.http.PushBuilder
    - Spring5, http2에서 사용할 수 있다.
    - 리소스 푸쉬에 사용한다.
    - 브라우저가 요청을 하지않아도 서버가 능동적으로 리소스를 푸쉬하는용도
    - 요청 -> 응답의 사이클이 줄어들기때문에 조금이나마 응답속도가 빨라진다.

- org.springframework.http.HttpMethod
    - 요청의 Method가 어떤 Method인지 정보를 가지고 있다.

- java.util.Locale, java.util.TimeZone, java.time.ZoneId
    - Spring MVC가 LocaleResolver Interface를 사용해서 분석한 Locale정보를 가지고 있다.

- @PathVariable
    - URI Template 변수를 읽을때 사용한다.

- @MatrixVariable
    - URI 경로중 키/값 쌍을 읽어올때 사용한다.

- @RequestBody
    - Http 본문에 있는 메시지를 HttpMessageConverter를 사용해서 읽어올 수 있다.

- @RequestParam
    - 서블릿 요청 매개변수값을 선언한 메서드 아규먼트 타입으로 변환해준다.
    - 단순 타입의경우 이를 생략할 수 있다.

- 그외 ..
    - https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/web.html#mvc-ann-methods

#### ReturnType

- @ResponseBody
    - 응답본문에 메시지를 보내고싶을때 사용

- HttpEntity, ResponseEntity
    - 응답본문 뿐 아니라, 헤더정보까지 전체 응답을 만들때 사용한다.
    - RESTAPI를 좀더 심도있게 만들려면 사용해야한다.

- String
    - ViewResolver를 사용해서 View를 찾을때 사용한다.
    - @ResponseBody를 사용중이라면 문자열을 요청본문에 직접 넣어준다.

- Map, Model
    - Model정보를 넣어준다.
    - RequestToNameTranslator를 사용해서 View를 요청에서 유추해서 ViewResolver를 찾는다.

- 그외 ..
    - 
