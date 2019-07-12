# Spring MVC WebMvcConfigurer
- EnableWebMvc는 Delegation 구조로 되어있다.
    - 원하는대로 확장이 가능한 구조
    - Interface형태로 지원

- WebMvcConfigurer
    - EnableWebMvc의 Bean을 사용하면서 커스터마이징하는 효과를 가진다.
    - Spring 3.1 version 부터 지원
    - Spring Boot 사용시에도 활용이 가능하다.


```java
package me.june;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ViewResolverRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-07
 * Time: 23:14
 **/
@Configuration
@ComponentScan(excludeFilters = {@ComponentScan.Filter(Service.class), @ComponentScan.Filter(Repository.class)})
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    // InternalResourceViewResolver를 확장
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF", "jsp");
    }
}
```

- ContentsNegotiationViewResolver
    - 클라이언트가 원하는 형태의 응답을 만드는 ViewResolver
    - Accept-Header를 이용하여 요청을 보낸다.
    - HTML, XML, JSON ... 
    - Spring-Boot의 경우에는 기본적으로 설정이 되어있다.

- 정리
    - Spring MVC프로젝트의 구조
        - Web.xml 혹은 WebApplicationInitializer를 구현하는 설정에서 DispatcherServlet을 등록한다.
        - ApplicationContext 설정 class 에서 @EnableWebMvc 를 사용하고, WebMvcConfigurer 를 구현하는 형태이다.

