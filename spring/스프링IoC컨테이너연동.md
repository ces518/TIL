# 스프링 IoC 컨테이너 연동
- 스프링을 사용하는 방법은 크게 두가지로 나뉜다.
    - 서블릿에서 스프링 IoC컨테이너를 활용하는방법
    - 스프링 MVC를 사용하는 방법

- 스프링 IoC컨테이너를 활용하는 방법
    - org.springframework.web.context.ContextLoaderListener
        - 스프링 IoC컨테이너를 서블릿 애플리케이션에 맞춰서 사용할수있도록 애플리케이션 컨텍스트를 등록해주는 역할
        - 서블릿 종료시 애플리케이션 컨텍스트 제거
    - 기본은 xml 기반 설정이지만 javaConfig로 바뀌는 추세

- web.xml
    - javaConfig를 사용할것이기 때문에
    - context-param으로 contextClass, org.springframework.web.context.support.AnnotationConfigWebApplicationContext 를 설정해준다.
    - AnnotationConfigWebApplicationContext는 ApplicationContext의 구현체중 하나로 애노테이션 기반 설정이 가능한 ApplicationContext이다
    - 추가적으로 contextParam으로 contextConfigLocation, me.june.AppConfig 를 등록해준다.
    - 설정 파일의 위치, 즉 javaConfig기반의 ApplicationContext가 될 클래스를 지정해준다.

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
  </context-param>

  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>me.june.AppConfig</param-value>
  </context-param>

  <filter>
    <filter-name>MyFilter</filter-name>
    <filter-class>me.june.MyFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>MyFilter</filter-name>
    <url-pattern>/hello</url-pattern>
  </filter-mapping>

  <servlet>
    <servlet-name>HelloServlet</servlet-name>
    <servlet-class>me.june.HelloServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>

```

- ContextLoader.class 의 initWebApplicationContext 메서드 내에서 webApplicationContext를 초기화하는 작업을 진행하는데
- WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE 의 이름으로 ApplicationContext를 등록해준다.

- AppConfig
```java
package me.june;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-07
 * Time: 23:14
 **/
@Configuration
@ComponentScan
public class AppConfig {
}

```
- SampleService
```java
package me.june;

import org.springframework.stereotype.Service;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-07
 * Time: 23:15
 **/
@Service
public class SampleService {

    public String getName () {
        return "juneyoung";
    }
}

```

- ApplicationContext를 사용하기 전에 SampleService를 빈으로 등록해서 ServlerApplication에서 사용하도록 해보자.

```java
package me.june;

import org.springframework.context.ApplicationContext;
import org.springframework.web.context.WebApplicationContext;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-04
 * Time: 21:57
 **/
public class HelloServlet extends HttpServlet {

    @Override
    public void init() throws ServletException {
        System.out.println("init");
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("doGet");

        /* Spring ApplicationContext에 빈으로 등록된 SampleService를 가져와 getName 활용하여 name을  출력 */
        ApplicationContext applicationContext = (ApplicationContext) getServletContext().getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
        SampleService sampleService = applicationContext.getBean(SampleService.class);

        PrintWriter writer = resp.getWriter();
        writer.println("<html>");
        writer.println("<head>");
        writer.println("</head>");
        writer.println("<body>");
        writer.println("<h1>");
        writer.println(String.format("Hello %s", sampleService.getName()));
        writer.println("</h1>");
        writer.println("</body>");
        writer.println("</html>");
    }
}
```

- DispatcherServlet
    - 스프링 MVC의 핵심
    - Front Controller 의 역할을 한다.


- 지금 등록한 ApplicationContext는 Root ApplicationContext이다.
- DispatchServlet도 ApplicationContext를 생성한다.
- Root ApplicationContext가 존재한다면 부모로 상속받아서 생성한다.
- Root ApplicationContex 는 여러 ApplicationContext에서 공유하여 사용할수 있다.
- DispatchServlet이 생성한 ApplicationContext는 해당 Servlet내에서만 scope이 제한된다.

- Root ApplicationContext는 여러 ApplicationContext에서 공유해야할 빈들 즉 Service, Repository 같은 객체를 등록하고
- DispatcherServlet의 ApplicationContext에는 Web과 관련있는 빈들 (Controller, ViewResolver 등 .. )을 등록한다.
