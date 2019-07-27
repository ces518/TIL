# Spring Handler Method - RequestParam
- 요청 매개변수를 Handler Argument로 받아오는 방법

- 요청 매개변수란 ?
    - 요청 매개변수는 크게 2가지로 분류된다.
    - 1. key/value 형식 URL Parameter (쿼리스트링) 
    - 2. HTTP 요청본문에 실어 보내는 formData 
- QueryParamter로 들어오든 , FormData로 넘어오든 같은방식으로 처리가 가능하다.

- @RequestParam
    - 요청 매개변수에 들어있는 단순 타입 데이터를 Method Argument로 받아올수있다.
    - 값이 반드시 있어야한다 (기본값 requried=true)
    - Optional을 지원한다.
    - String이 아닌 타입은 Type-Conversion을 지원한다.
    - Map<String, String> 또는 MultiValueMap<String, String>에 사용해서 모든 요청 매개변수를 받아올 수 있다.
    - 생략이 가능하다.
    - 기본값 지정이 가능하다.

- @RequestParam.java
```java
/*
 * Copyright 2002-2018 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.Map;

import org.springframework.core.annotation.AliasFor;

/**
 * Annotation which indicates that a method parameter should be bound to a web
 * request parameter.
 *
 * <p>Supported for annotated handler methods in Spring MVC and Spring WebFlux
 * as follows:
 * <ul>
 * <li>In Spring MVC, "request parameters" map to query parameters, form data,
 * and parts in multipart requests. This is because the Servlet API combines
 * query parameters and form data into a single map called "parameters", and
 * that includes automatic parsing of the request body.
 * <li>In Spring WebFlux, "request parameters" map to query parameters only.
 * To work with all 3, query, form data, and multipart data, you can use data
 * binding to a command object annotated with {@link ModelAttribute}.
 * </ul>
 *
 * <p>If the method parameter type is {@link Map} and a request parameter name
 * is specified, then the request parameter value is converted to a {@link Map}
 * assuming an appropriate conversion strategy is available.
 *
 * <p>If the method parameter is {@link java.util.Map Map&lt;String, String&gt;} or
 * {@link org.springframework.util.MultiValueMap MultiValueMap&lt;String, String&gt;}
 * and a parameter name is not specified, then the map parameter is populated
 * with all request parameter names and values.
 *
 * @author Arjen Poutsma
 * @author Juergen Hoeller
 * @author Sam Brannen
 * @since 2.5
 * @see RequestMapping
 * @see RequestHeader
 * @see CookieValue
 */
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestParam {

	/**
	 * Alias for {@link #name}.
	 */
	@AliasFor("name")
	String value() default "";

	/**
	 * The name of the request parameter to bind to.
	 * @since 4.2
	 */
	@AliasFor("value")
	String name() default "";

	/**
	 * Whether the parameter is required.
	 * <p>Defaults to {@code true}, leading to an exception being thrown
	 * if the parameter is missing in the request. Switch this to
	 * {@code false} if you prefer a {@code null} value if the parameter is
	 * not present in the request.
	 * <p>Alternatively, provide a {@link #defaultValue}, which implicitly
	 * sets this flag to {@code false}.
	 */
	boolean required() default true;

	/**
	 * The default value to use as a fallback when the request parameter is
	 * not provided or has an empty value.
	 * <p>Supplying a default value implicitly sets {@link #required} to
	 * {@code false}.
	 */
	String defaultValue() default ValueConstants.DEFAULT_NONE;

}
```

- name 파라메터를 받아서, Event 객체를 생성후, 해당 객체를 Return 하는 Handler코드 작성
```java
@Controller
public class MvcController {

    @GetMapping("/mvc/events")
    @ResponseBody
    public Event hello (@RequestParam String name) {
        Event events = new Event();
        events.setName(name);
        return events;
    }

}
```

- 테스트 코드
    - GET /mvc/events?name=june 으로 요청을 보내는 테스트 코드
```java
@Test
public void getEvent () throws Exception {
    this.mockMvc.perform(get("/mvc/events")
                    .param("name", "june"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("june"));
}
```

- 테스트 결과
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /mvc/events
       Parameters = {name=[june]}
          Headers = []
             Body = <no character encoding set>
    Session Attrs = {}

Handler:
             Type = me.june.springbootmvc.mvc.MvcController
           Method = public me.june.springbootmvc.Event me.june.springbootmvc.mvc.MvcController.hello(java.lang.String)

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
          Headers = [Content-Type:"application/json;charset=UTF-8"]
     Content type = application/json;charset=UTF-8
             Body = {"id":null,"name":"june","starts":null}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
