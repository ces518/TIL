# Spring Boot - MVC 설정
- Spring boot의 주관에 따라 기본적으로 Bean이 등록된다.


- HandlerMapping
    - SimpleUrlHandlerMapping (favicon 요청처리)
    - RequestMappingHanlderMapping (애노테이션 기반 MVC)
    - BeanNameUrlHandlerMapping 
    - SimpleUrlHandlerMapping (resouceHandlerMapping_정적리소스제공 기능) 
        - 응답헤더에 캐시정보를 추가해준다.
        - 캐시정보를 기반으로 Resource를 효율적으로 제공한다.
        - Resource가 변경되지 않았을경우 Not Modified 응답을 보내줌으로써 브라우저가 캐싱하고있는 Resource를 사용하도록 한다.
    - WelcomPageHandlerMapping


- HandlerAdapter
    - RequestMappingHandlerAdapter
    - HttpRequestHandlerAdapter
    - SimpleControllerHandlerAdapter


- ViewResolver
    - ContentNegotiatingViewResolver
        - 아래의 ViewResolver들이 ContentNegotiatingViewResolver이 Delegating하는 ViewResolver 들이다.
        - ContentNegotiatingViewResolver는 직접 View를 보내주는것이 아니다.
        - 다른 ViewResolver들을 내부적으로 참조하고있다.
    - BeanNameViewResolver
    - ThymeleafViewResolver
    - ViewResolverComposite
    - InternalResourceViewResolver


- 자동설정
    - org.springframework.boot:spring-boot-autoconfigure
    - spring boot 가 제공하는 자동설정 의존성
        - spring.factories에 다양한 자동설정 클래스들이 명시되어있다
        - 특정 조건에 따라 빈으로 등록된다.


- DispatcherServletAutoConfiguration
    - DispatcherServlet을 특정 설정에 따라 자동적으로 등록해준다.


- WebMvcAutoConfiguration
    - WebMvc관련 자동 설정
    - Servlet Type의 애플리케이션일경우 웹 애플리케이션 관련 자동 설정을 해준다.


- @ConditionalOnClass
    - 해당 클래스가 클래스패스엥 존재할경우 설정을 적용하도록 한다.


- @ConditionalOnMissingBean
    - 해당 클래스가 빈으로 등록되어 있지않을경우 설정을 적용하도록한다.


- Spring boot properties
    - Spring boot 의 자동설정은 properties를 읽어와서 적용되는 설정
    - 즉 properties 설정파일에서 커스터마이징이 가능하다.


- Spring boot MVC 커스터마이징
    - application.properties (스프링부트의 자동설정 + 자동설정 커스터마이징)
    - @Configuration + WebMvcConfigurer (스프링부트의 자동설정 + 확장 설정)
    - @Configuration + @EnableWebMvc + WebMvcConfigurer (Spring boot 자동설정을 사용하고 싶지 않을경우 사용하는 방법)
