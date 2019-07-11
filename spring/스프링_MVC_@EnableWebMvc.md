# Spring MVC - @EnableWebMvc
- 애노테이션 기반 Spring MVC를 사용할때 편리한 웹 MVC 기본 설정 방법

- DelegatingWebMvcConfiguration
    - EnableWebMvc 애노테이션을 사용하면 DelegatingWebMvcConfiguration 을 Import 한다.
    - DelegatingWebMvcConfiguration 은 WebMvcConfigurationSupport를 상속받고있다.
    - 다양한 Bean 설정이 등록 되어있다.

- WebApplication.java
    - EnableWebMvc 를 사용하기 이전에 추가적인 설정이 필요하다.
    - EnableWebMvc 에서 ServletContext를 종종 참조하기때문에
    - ApplicationContext에 ServletContext를 등록해주어야 제대로 동작한다.

```java
package me.june;

import org.springframework.web.WebApplicationInitializer;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRegistration;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-11
 * Time: 20:31
 **/
public class WebApplication implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        AnnotationConfigWebApplicationContext applicationContext = new AnnotationConfigWebApplicationContext();
        // EnableWebMvc 사용시 필수 
        applicationContext.setServletContext(servletContext);
        // WebConfig.class를 설정 클래스로 지정
        applicationContext.register(WebConfig.class);
        applicationContext.refresh();

        // DispatcherServlet을 등록
        DispatcherServlet dispatcherServlet = new DispatcherServlet(applicationContext);
        ServletRegistration.Dynamic app = servletContext.addServlet("app", dispatcherServlet);
        app.addMapping("/app/*");
    }
}
```

- WebConfig.java
    - @EnableWebMvc 사용

```java
package me.june;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-07
 * Time: 23:14
 **/
@Configuration
@ComponentScan(excludeFilters = {@ComponentScan.Filter(Service.class), @ComponentScan.Filter(Repository.class)})
@EnableWebMvc
public class WebConfig {

    @Bean
    public ViewResolver viewResolver () {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```

- EnableWebMvc를 사용할 경우 Bean 설정
    - HandlerMapping
        - RequestMappingHandlerMapping의 우선순위가 더 높아진다.
    - Interceptors
        - 기존에는 없었던 2개의 interceptor가 등록된다.
        - ConversionServiceExposingInterceptor
            - spring:eval 태그를 jsp에서 사용할수있도록 해줌
        - ResourceUrlProviderExposingInterceptor
    - HandlerAdapter
        - RequestMappingHandlerAdapter 의 우선순위가 더 높아진다.
    - ...

