# Spring MVC - HandlerInterceptor 구현 및 등록
- 간단한 인터셉터 구현

- HelloInterceptor.java
```java
package me.june.springbootmvc;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-16
 * Time: 22:43
 **/
public class HelloInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle 1");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle 1");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion 1");
    }
}
```

- SimpleInterceptor
```java
package me.june.springbootmvc;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-16
 * Time: 22:43
 **/
public class SimpleInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle 2");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle 2");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion 2");
    }
}
```

- 구현한 인터셉터 등록
- WebConfig.java
    - Order를 주지않았다면, 등록된 순서대로 우선순위가 지정된다.
```java
package me.june.springbootmvc;

import org.springframework.context.annotation.Configuration;
import org.springframework.format.FormatterRegistry;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-14
 * Time: 21:25
 **/
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new PersonFormatter());
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new HelloInterceptor());
        registry.addInterceptor(new SimpleInterceptor());
    }
}
```

- Interceptor 실행결과
    - preHandle: 호출의 정순
    - postHandle: 호출의 역순
    - afterCompletion: 호출의 역순
```java
preHandle 1
preHandle 2
postHandle 2
postHandle 1
afterCompletion 2
afterCompletion 1
```

- Interceptor적용시 Order를 사용하여 우선순위를 지정해 줄수있다.
```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new HelloInterceptor()).order(0);
    registry.addInterceptor(new SimpleInterceptor()).order(-1);
}
```

#### 정리   
- Interceptor생성시 HandlerInterceptor Interface를 구현하여 Interceptor를 작성할 수 있다.
- 생성한 Interceptor를 등록하고싶다면 WebMvcConfigurer Interface의 addInterceptors메서드를 통해 등록이 가능하다.
- Interceptor등록시 우선순위를 지정해 줄 수 있으며, 우선순위를 지정하지 않았다면 등록한 순서대로 우선순위가 지정된다.
    - 이때 우선순위는 음수가 우선순위가 더 높다.
- Interceptor등록시 특정 Url Pattern의 요청시에만 동작하도록 등록도 가능하다.
