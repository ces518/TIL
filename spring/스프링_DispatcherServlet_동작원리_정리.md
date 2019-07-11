# Spring MVC 동작원리 - 정리
- 서블릿 기반 애플리케이션이고, 서블릿 컨테이너가 필요하다.

- DispatcherServlet 초기화
    - 1. 특정 타입에 해당하는 빈을 찾는다.
    - 2. 해당 빈이 없다면 기본 전략을 사용한다. DispatcherServlet.properties

- Spring MVC
    - 서블릿 컨테이너에 등록한 웹 애플리케이션에 DispatcherServlet을 등록해서 사용하는 구조
    - 세부 구성요소는 설정하기 나름

- Spring Boot
    - 자바 애플리케이션 내부에 내장 톰캣을 만들고 그 안에 DispatcherServletd이 존재하는 구조
        - Spring boot 자동설정
    - Spring boot의 기본 설정에 따라 여러 인터페이스 구현체를 빈으로 등록한다.


- web.xml 없이 서블릿 애플리케이션 등록방법
    - Servlet3.0, Spring 3.1 이상에서 지원하는 방법
    - org.springframework.web.WebApplicationInitializer를 구현한다.
    - javaCode로 ApplicationContext와 DispatcherServlet을 등록해서 사용하는 방법.
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
