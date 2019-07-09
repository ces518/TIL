# DispatcherServlet 동작원리 3
- DispatcherServlet 기본 설정 전략
  - 대부분의 전략의 구동 방식이 비슷하다.
  - ViewResolver 전략만 살펴보도록 하자.


- 기본 설정 전략
  - DispatcherServlet.properties


- initViewResolvers
  - viewResolver 기본 설정 전략
  - 기본적으로 빈으로 등록된 Resolver를 찾아오는데 없다면 ?
  - 기본전략에 따라 InternalResourceViewResolver 를 사용한다.


- WebConfig.class
  - ViewResolver로 InternalResourceViewResolver를 커스터마이징해서 등록한다. (기본 설정 전략에 사용되는 ViewResolver와 동일함.)

```java
package me.june;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-07
 * Time: 23:14
 **/
@Configuration
@ComponentScan(excludeFilters = {@ComponentScan.Filter(Service.class), @ComponentScan.Filter(Repository.class)})
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

- InternalResourceViewResolver
  - Prefix: View네임에 공통으로 사용될 prefix 부분에 해당
  - Suffix: View네임에 공통으로 사용될 suffix 부분에 해당


- HelloController.class
  - 기존 ViewResolver는 prefix suffix에 해당하는부분이 정의되지 않았지만
  - InternalResourceViewResolver를 커스터마이징 했기때문에
  - prefix와 suffix부분에 해당하던 viewname에서 사라지게 되었다.

```java
package me.june;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-08
 * Time: 20:40
 **/
@Controller
public class HelloController {

    @Autowired
    private SampleService sampleService;

    @GetMapping("/hello")
    @ResponseBody
    public String hello () {
        return "Hello " + sampleService.getName();
    }

    @GetMapping("/sample")
    public String sample () {
        // /WEB-INF/sample.jsp
        return "sample";
    }
}
```
