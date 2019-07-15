# Spring MVC - Handler Interceptor
- HandlerInterceptor
    - HandlerMapping 에 설정할 수 있는 인터셉터
    - 핸들러를 실행하기 전, 후 그리고 완료 시점에 부가작업을 하고싶은경우 사용할 수 있다.
    - 여러 핸들러에서 반복적으로 사용하는 코드를 줄이고 싶을때 사용가능하다.
        - 로깅, 인증 등 ...

```java
// preHandle
// 요청 처리
// postHandle
// 뷰 렌더링
// afterCompletion
```

- boolean preHandle(req, res, handler)
    - handler 실행 전 호출
    - handler 의 정보를 사용할 수 있기 때문에 서블릿 필터보다 세밀한 로직을 구현할 수 있다.
    - 리턴타입 (boolean) 으로 다음 인터셉터 or 핸들러로 요청, 응답 처리를 진행할것인지 처리가 가능하다.

- void postHandle(req, res, modelAndView)
    - 핸들러 실행이 종료되고, 뷰를 렌더링 하기 이전에 호출
    - 뷰에 전달할 추가적이거나 여러 핸들러에 공통적인 모델정보를 담는데 사용할 수 있다.
    - 이 메서드는 인터셉터의 역순으로 호출된다.
    - '비동기적 요청 처리시' 에는 호출되지 않는다.

- void afterCompletion(request, response, handler, ex)
    - 요청처리가 완전히 끝난뒤 호출된다
    - preHandle에서 true를 리턴한 경우에만 호출된다.
    - 이 메서드는 인터셉터의 역순으로 호출된다.
    - 비동기적 요청 처리시에는 호출되지 않는다.

### 정리
- 서블릿 보다 구체적인 처리가 가능하다.
- 서블릿은 보다 일반적인 용도의 기능을 구현하는데 사용하는것이 좋다.
