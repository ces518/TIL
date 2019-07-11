# Spring MVC 빈 설정
- Spring MVC를 사용하면 특정한 설정이 없는한 기본 전략에 따라 interface 구현체들이 빈으로 등록된다.

- 해당 설정을 변경 하거나, 추가적인 설정 추가하고 싶은경우 다음과 같이 빈으로 등록이 가능하다.

- WebConfig.java
    - 기본 전략에 따라 생성된 빈들은 다음과 같이 new 로 등록된것과 동일하다.
    - DispatcherServlet기본전략에 따라 등록된 RequestMappingHandlerMapping 은 다음의 코드와 동일하다.
    - Bean으로 등록할경우 추가적인 설정을 더하는 등 커스터마이징이 가능하다.
```java
package me.june;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
import org.springframework.web.servlet.HandlerMapping;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
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
    public HandlerMapping handlerMapping () {
        RequestMappingHandlerMapping handlerMapping = new RequestMappingHandlerMapping();
        handlerMapping.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return handlerMapping;
    }

    @Bean
    public ViewResolver viewResolver () {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```

- 가장 Low Level의 설정방법이다.

- ArgumentResolver
    - Handler Method의 Argument로 사용할수있는 것들은 다양하다
    - Pathvariable
    - RequestParam
    - ModelAttribute

- MessageConverter
    - 요청본문의 값을 파라메터로 바인딩하거나, 응답을 요청본문으로 보낼때 사용된다.
