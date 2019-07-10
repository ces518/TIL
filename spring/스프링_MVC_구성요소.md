# Spring MVC 구성요소
- DispatcherServlet이 init lifeCycle에 의해 initStrategies 메서드가 호출된다.
- initStrategies 메서드에는 다양한 인터페이스들의 초기화 메서드가 존재한다.
    - initMultipartResvoler
    - initLocaleResolver
    - initThemeResolver
    - initHandlerMappings
    - initHandlerAdapters
    - initHandlerExceptionResolvers
    - initRequestToViewNameTranslator
    - initViewResolvers
    - initFlashMapManager

- 위 9가지 가 DispatacherServlet을 구성하는 핵심 인터페이스들이다.


- MulitpartResolver
    - 요청 분석단계에서 사용된다.
    - 파일업로드 처리를 하는 Interface
    - MultipartResolver Type의 Bean이 등록되어 있어야하고, DispathcerServlet이 해당 Bean을 가지고 있어야 파일업로드 요청을 처리할 수 있다.
    - HttpServletRequest > MultipartHttpServletRequest로 변환해준다.
    - Http 요청으로 같이 넘어온 File을 MultipartFile객체로 파일을 핸들링 할 수 있다.
    - CommonsMultipartResolver, StandardServetMultipartResolver 두 개의 구현체가 존재하고, StandardServetMultipartResolver는 Servlet 3.0 기반이다.
    - Spring MVC 전략에는 MultipartResolver를 등록하지않는다.
    - Spring Boot는 MultipartResolver를 기본적으로 등록해주기때문에 특별한 설정 없이도 파일업로드 처리를 할 수 있다.


- LocaleResolver
    - 요청 분석단계에서 사용된다.
    - Client의 지역 정보를 확인하는데 사용된다. 
    - 해당 정보를 기반으로 MessageSource에서 적절한 Message를 응답한다.
    - 구현체
        - 기본 전략은 AcceptHeaderLocaleResolver를 Bean으로 등록한다.
        - AcceptHeaderLocaleResolver
        - CookieLocaleResolver
        - FixedLocaleResolver
        - LocaleContextResolver
        - SessionLocalResolver


- ThemeResolver
    - Spring MVC는 Theme기능을 제공한다.
    - 웹 브라우저에서 특정 버튼을 눌렀을때 테마가 변경되는 기능을 제공하는경우
    - Theme 정보를 View로 전달하고 해당 Theme와 일치하는 적절한 CSS를 읽어와서 Theme 변경가능하다.
    - 구현체
        - 기본 전략은 FixedThemeResolver를 Bean으로 등록한다.
        - CookieThemeResolver
        - FixedThemeResolver
        - SessionThemeResolver


- HandlerMapping
    - 요청이 들어 왔을때 해당 요청을 처리할수 있는 핸들러를 찾아주는 인터페이스
    - 해당 요청을 수행할 수 있는 메서드정보를 가지고있는 Handler객체를 리턴함.
    - 구현체
        - 기본 전략 으로 아래 두 구현체를 빈으로 등록한다.
        - BeanNameUrlHandlerMapping
            - Controller interface 구현하고있는 핸들러를 찾는다.
        - RequestMappingHandlerMapping
            - 애노테이션 기반 핸들러를 찾는다.


- HandlerAdapter
    - 각각의 핸들러를 처리할 수 있는 interface
    - Spring MVC 확장력의 핵심
    - 구현체
        - RequestMappingHandlerAdapter
            - 애노테이션 기반 핸들러를 처리할수 있다.
        - SimpleControllerHandlerAdapter
            - Controller interface 를 구현하고있는 핸들러를 처리할 수 있다.
        - HttpRequestHandlerAdapter


- HandlerExceptionResolver
    - 요청 처리중 발생한 예외를 처리하는 interface
    - 구현체
        - ResponseStatusExceptionResolver
        - ExceptionHandlerExceptionResolver
        - DefaultHandlerExceptionResolver

- RequestToViewNameTranslator
    - Handler에서 응답 View로 만들어서 보낼때 일반적으로 ViewName을 명시하지만 해당 ViewName을 생략할 수도 있다.
    - ViewName에 대한 정보가 없을경우 요청을 기반으로 ViewName을 결정한다.
    - 구현체
        - DefaultRequestToViewNameTranslator
```java
@GetMapping("/sample")
public void sample () {}
```

- ViewResolver
    - ViewName에 해당하는 뷰를 찾아내는 interface
    - 구현체
        - InternalResourceViewResolver
        - ...


- FlashMapManager
    - FlashMap 인스턴스를 가져오고, 저장하는 interface
    - 주로 Redirection을 사용할때 요청 매개변수를 사용하지않고 데이터를 전달하고, 정리할때 사용된다.
    - 화면이 Refresh되었을때 같은 정보를 다시 받아오지 않도록 하는 패턴
    - 구현체
        - SessionFlashMapManager
