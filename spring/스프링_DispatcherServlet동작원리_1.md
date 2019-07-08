# DispatcherServlet의 동작 원리 1

- DispatcherServlet 초기화
  - 특별한 타입의 빈들을 찾거나, 기본 전략에 해당하는 빈들을 등록한다.
  - HandlerMapping
  - HandlerAdapter
  - HandlerExceptionResolver
  - ViewResolver
  - ...

- HandlerMapping
  - 핸들러를 찾아주는 인터페이스
  - 아무런 설정을 하지않아도 DispatcherServlet이 등록해주는 핸들러가 2개가 존재한다.
    - BeanNameUrlHandlerMapping
    - * RequestMappingHandlerMapping (@RequestMapping..)
  - 서치 전략
    - 1. BeanNameUrlHandlerMapping
    - 2. RequestMappingHandlerMapping
    - 1 -> 2 순으로 핸들러를 찾는다.
  - 대부분이 전략 패턴이 적용되어있다.

- HandlerAdapter
  - 핸들러를 실행하는 인터페이스
  - 기본으로 등록되어있는 HandlerAdapter가 3개 존재한다.
    - HttpRequestHandlerAdapter
    - SimpleControllerHanlderAdapter
    - * RequestMappingHanlderAdapter
  - 서치 전략
    - 1. HttpRequestHandlerAdapter
    - 2. SimpleControllerHanlderAdapter
    - 3. RequestMappingHanlderAdapter
    - 1 -> 2 -> 3 순으로 HandlerMapping이 찾은 Handler를 사용할수 있는 HandlerAdapter를 찾는다.

- 실행 하기 이전에 HttpMethod가 GET인지 체크,  캐싱기능을 제공할지 판단한다.
- 그 후 javaReflection을 활용해서 해당 핸들러 메서드를 호출한다.
- returnValueHanlder를 통해서 return값을 처리할 핸들러를 찾는다.
- 15가지가 존재함...
  - ModelAndViewMethodReturnValueHandler
  - ModelMethodProcessor
  - ViewMethodReturnValueHandler
  - ReponseBodyEmitterReturnValueHandler
  - StreamingResponseBodyReturnValuneHandler
  - HttpEntityMethodProcessor
  - HttpHeadersReturnValueHandler
  - CallableMethodReturnValueHandler
  - DeferredResultMethodReturnValueHandler
  - AsyncTaskMethodReturnValueHandler
  - ModelAttributeMethodProcessor
  - RequestResponseBodyMethodProcessor
  - ViewNameMethodReturnValuneHandler
  - MapMethodProcessor
  - ModelAttributeMethodProcessor

- * RestController는 일반적인 Controller에 ResponseBody를 적용한 것과 동일하다.

#### 정리
- DispatcherServlet 동작 순서
- 1. 요청을 분석한다.
- 2. 요청을 처리할 핸들러를 찾는다
- 3. 핸들러를 실행할 수 있는 어댑터를 찾는다.
- 4. 핸들러 어댑터를 사용해서 응답을 처리한다.
- 5. 예외가 발생했다면, 예외처리 핸들러에게 위임한다.
- 6. 핸들러의 리턴값을 보고 어떻게 처리핡서인지 판단한다.
  - 뷰 이름에 해당하는 뷰를 찾아 응답 페이지를 랜더링
  - @ResponseEntity 가 존재한다면 , Converter를 사용하여 응답 본문을 만든다.
- 7. 최종적으로 응답을 클라이언트에게 보낸다.
