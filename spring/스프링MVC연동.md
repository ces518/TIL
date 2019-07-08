# 스프링 MVC 연동
- DispatcherServlet을 활용해서 스프링 MVC 를 사용 하는 방법

- DispatchServlet
  - service 와 repository 같이 web과 관련없는 빈들을 가지는 root Context를 상속받는 applicationContext를 가진다.
  - 즉 Root Context에서 등록한 service, repository와 같은 빈들을 사용할 수 있으며, 독립적인 context를 가진다.
  - 주로 dispatcherServlet 단위로 web과 관련된 빈들을 등록한다.

- DispatcherServlet 등록
  - app이라는 이름으로 DispatcherServlet을 등록한다.
  - ContextLoaderListener와 마찬가지로 AnnotationConfigWebApplicationContext를 ApplicationContext의 구현체로 사용한다.
  - 해당 컨텍스트의 설정 클래스를 me.june.WebConfig 로 지정한다.
  - /app/** 으로 시작하는 요청은 모두 DispatcherServlet이 처리한다.

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
  </context-param>

  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>me.june.AppConfig</param-value>
  </context-param>

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <servlet>
    <servlet-name>app</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextClass</param-name>
      <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
    </init-param>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>me.june.WebConfig</param-value>
    </init-param>
  </servlet>

  <servlet-mapping>
    <servlet-name>app</servlet-name>
    <url-pattern>/app/*</url-pattern>
  </servlet-mapping>

</web-app>
```

- Root Context 변경 
  - 기본적으로 ComponentScan시 모든 Component 애노테이션을 빈으로 등록하는데 Root Context이기 때문에 Controller를 제외한 클래스
  - 즉 웹과 관련이없는 Servce, Repository 등의 클래스만 빈으로 등록하도록 설정을 변경한다.

```java
package me.june;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-07
 * Time: 23:14
 **/
@Configuration
@ComponentScan(excludeFilters = @ComponentScan.Filter(Controller.class))
public class AppConfig {
}

```

- Servlet단위의 ApplicationContext 설정
  - Servlet단위의 ApplicationContext는 앞서 말했듯이 RootContext를 상속받는다.
  - 즉 Service, Repository는 RootContext에 빈으로 등록되어있는 상태이고, 이를 상속받는 context에서 사용할수 있기때문에 빈 설정에서 제외한다.
  - Controller와 같은 웹과 관련있는 빈만 등록한다. 

```java
package me.june;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-07
 * Time: 23:14
 **/
@Configuration
@ComponentScan(excludeFilters = {@ComponentScan.Filter(Service.class), @ComponentScan.Filter(Repository.class)})
public class WebConfig {
}
```

- 설정 결과
  - ContextLoaderListener에 의해 서블릿 애플리케이션이 구동될때 RootContext가 생성된다.
  - 해당 Context의 설정 클래스는 AppConfig.class 이며, Web과 관련없는 빈만 등록이 된다.
  - DispatcherServlet을 Servlet으로 등록하고, Context 설정 클래스 (WebConfig.class) 에 의해 Web과 관련있는 빈을 가지는 Context가 생성된다.
  - 이후 요청은 /app/** 를 DispatcherServlet에 등록된 핸들러가 요청을 처리하게된다.

- 계층구조를 사용하고 싶지않다면, DispatcherServlet의 ApplicationContext에 모든 빈을 등록해도 무관하다.
- 최근에는 대부분의 프로젝트가 DispatcherServlet 하나만 사용하고, 해당 ApplicationContext에 모든 빈을 등록한다.

- 스프링 부트와는 다른 구조이다.
  - 스프링 MVC는 톰캣안에 스프링이 들어가는 구조
  - 스프링 부트는 Servlet 애플리케이션 내부에 톰캣이 들어가고 톰캣 내부에 DispatcherServlet 이 들어가는 구조
