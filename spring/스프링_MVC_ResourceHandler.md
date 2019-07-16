# Spring MVC - Resource Handler
- ResourceHanlder 란 ?
    - Servlet Container 가 제공하는 DefaultServlet에 대한 이해가 필요하다.
    - Image, JS, CSS HTML 과 같은 정적 리소스를 처리하는 Handler 이다.
    - 모든 톰캣에는 이러한 정적 리소스를 처리할 수 있는 DefaultServlet이 등록 되어있다.
    - Directory Listing 용도로도 사용된다.
    - $CATALINA_BASE/conf/web.xml 에 전역적으로 등록되어있음.
    - Spring 은 DefaultServlet에 요청을 '위임' 해서 이러한 Resource 요청을 처리하는것이다.
    - ResourceHandler들은 우선순위가 가장 낮다.
    - Spring Boot 사용시 기본 설정에 의해 ResouceHandler 기능을 제공받는다.

- classpath:resources/static/index.html 파일이 존재할때
- /index.html 로 요청을 보내면 classpath:resources/static/index.html 파일을 응답으로 받을 수 있다.

- index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello<title>
</head>
<body>
<h1>Hello Spring Boot ! </h1>
</body>
</html>
```

- http://localhost:8080/index.html 로 요청을 보내면 Hello Spring Boot! 응답이 오는것을 확인할 수 있다.

- 그외 추가적으로 ResourceHandler가 필요하다면 직접 등록할 수 있다.

- WebConfig.java
    - /spring/** 으로 시작하는 모든요청을 ResourceHandler가 처리하도록한다.
    - 해당 요청이 들어올때 제공할 Resource의 경로를 classpath:/spring/** 로 지정한다.
    - 즉 /spring/** 으로 시작하는 Resource요청은 classpath:/spring/** 폴더에 존재하는 Resource를 제공하게 된다.
    - 해당 리소스에 대한 cache설정도 가능하다.
    - CacheControl.maxAge(maxAge, TimeUnit);
        - 첫번째 인자로 maxAge를 받고, 두번째 인자로 TimeUnit, 시간의 단위를 받는다.
        - 아래의 설정에선 캐시설정을 10분으로 지정한것이다.
    - 해당 캐시 설정을 한다면, 기본적으로 캐시관련된 Header가 응답헤더에 추가가된다.
    - fileSystem 기준 경로로 설정도 가능하다.
```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/spring/**")
                .addResourceLocations("classpath:/spring/")
                .setCacheControl(CacheControl.maxAge(10, TimeUnit.MINUTES));
}
```

- 다음 설정에 대한 테스트코드

```java
package me.june.springbootmvc;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.HttpHeaders;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello () throws Exception {
        this.mockMvc.perform(get("/spring/index.html"))
                .andExpect(status().isOk())
                .andExpect(header().exists(HttpHeaders.CACHE_CONTROL))
                .andDo(print());
    }

}
```

- 테스트 결과
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /spring/index.html
       Parameters = {}
          Headers = []
             Body = <no character encoding set>
    Session Attrs = {}

Handler:
             Type = org.springframework.web.servlet.resource.ResourceHttpRequestHandler

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Last-Modified:"Tue, 16 Jul 2019 14:16:41 GMT", Cache-Control:"max-age=600", Content-Length:"152", Content-Type:"text/html", Accept-Ranges:"bytes"]
     Content type = text/html
             Body = <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Spring</title>
</head>
<body>
<h1>Hi, Spring Boot ! </h1>
</body>
</html>

    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

- Spring Boot가 기본으로 제공하는 ResouceHandler에 대한 캐시설정도 가능하다
- application.properties 에서 제어가 가능하다.
- Spring 4.1 이상
    - ResourceChain: 캐시 사용 유무
    - Transformer와 Resolver를 추가할 수 있다.
    - ResourceResolver 요청에 해당하는 리소스를 찾는 전략
        - 캐싱, 인코딩 (gzip) WebJar ...
            - Spring Boot를 사용한다면 기본적으로 제공하기때문에 직접 설정할 일이 없음. 
    - ResourceTransformer
        - 응답으로 보낼 리소르를 수정하는 전략
        - 캐싱, CSS 링크, HTML5 , Meta ... 


#### 정리
- ServletApplication을 개발한다면 기본적으로 Tomcat에 DefaultServlet이 존재한다.
- DefaultServlet은 HTML CSS JS 와 같은 정적 리소스요청에 대한 처리를 한다.
- Spring 은 ResourceHandler를 사용하여 이 정적 리소스 요청을 DefaultServlet으로 위임하여 요청을 처리한다.
- Spring Boot는 기본적인 ResourceHandler 가 등록되어있으며, 캐시설정은 application.properties를 통하여 제어가 가능하다.
- 추가적인 ResourceHandler가 필요하다면 WebMvcConfigurer Interface의 메서드를 통해 등록이 가능하다.
