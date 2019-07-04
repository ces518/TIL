# 서블릿 애플리케이션
- IntelliJ 사용, Maven 을 활용한 프로젝트생성
- 1. newProject > Maven > Create from archetype 체크
    - archetype 이란 ?
    - 메이븐에서 미리 만들어놓은 프로젝트 틀이라고 생각
- 2. org.apache.maven.archetypes:maven-archetype-webapp 선택
- 3. next 후 프로젝트 생성


- pom.xml
- servlet-api 를 의존성으로 추가
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

- scope 란 ? 
    - 이 의존성을 언제 classpath에 넣고 사용할것인지에 대한 scope
- provided
    - 런타임시 외부로부터 제공을 받기 때문에 런타임시 제외한다.
- test
    - test 시에만 사용이 가능하다.


### 코드 작성
- 간단한 서블릿 애플리케이션 코드 작성
```java
package me.june;

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
        PrintWriter writer = resp.getWriter();
        writer.println("<html>");
        writer.println("<head>");
        writer.println("</head>");
        writer.println("<body>");
        writer.println("<h1>");
        writer.println("Hello Servlet!!");
        writer.println("</h1>");
        writer.println("</body>");
        writer.println("</html>");
    }
}
```

- 서블릿을 독자적으로 실행할수 있는 방법은 없다.
- 반드시 서블릿컨테이너 위에서 구동되어야한다.
- 일반적으로 톰캣을 많이 사용한다.

- 서블릿을 web.xml에 등록하고, 서버를 실행해보자
- web.xml
```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
  
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

- 서블릿 애플리케이션이 정상적으로 동작하는지 확인
- http://localhost:8080/hello 로 요청
- console 출력확인
    - init
    - doGet
    - 순서로 로그가 찍힌다.
