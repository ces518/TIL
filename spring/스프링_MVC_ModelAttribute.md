# Spring Handler Method - @ModelAttribute
- @RequestParam과 같이 요청 매개변수를 매핑하는 방법중 하나이다.

- @ModelAttribute
    - 단순 데이터 타입을 하나의 복합타입의 객체로 받아오거나, 객체를 새로 생성할때 사용할수 있다.
    - URLPath, 요청매개변수, 세션 등 ..
    - 생략이 가능하다.
```java
/*
 * Copyright 2002-2016 the original author or authors.
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

import org.springframework.core.annotation.AliasFor;
import org.springframework.ui.Model;

/**
 * Annotation that binds a method parameter or method return value
 * to a named model attribute, exposed to a web view. Supported
 * for controller classes with {@link RequestMapping @RequestMapping}
 * methods.
 *
 * <p>Can be used to expose command objects to a web view, using
 * specific attribute names, through annotating corresponding
 * parameters of an {@link RequestMapping @RequestMapping} method.
 *
 * <p>Can also be used to expose reference data to a web view
 * through annotating accessor methods in a controller class with
 * {@link RequestMapping @RequestMapping} methods. Such accessor
 * methods are allowed to have any arguments that
 * {@link RequestMapping @RequestMapping} methods support, returning
 * the model attribute value to expose.
 *
 * <p>Note however that reference data and all other model content is
 * not available to web views when request processing results in an
 * {@code Exception} since the exception could be raised at any time
 * making the content of the model unreliable. For this reason
 * {@link ExceptionHandler @ExceptionHandler} methods do not provide
 * access to a {@link Model} argument.
 *
 * @author Juergen Hoeller
 * @author Rossen Stoyanchev
 * @since 2.5
 */
@Target({ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ModelAttribute {

	/**
	 * Alias for {@link #name}.
	 */
	@AliasFor("name")
	String value() default "";

	/**
	 * The name of the model attribute to bind to.
	 * <p>The default model attribute name is inferred from the declared
	 * attribute type (i.e. the method parameter type or method return type),
	 * based on the non-qualified class name:
	 * e.g. "orderAddress" for class "mypackage.OrderAddress",
	 * or "orderAddressList" for "List&lt;mypackage.OrderAddress&gt;".
	 * @since 4.3
	 */
	@AliasFor("value")
	String name() default "";

	/**
	 * Allows declaring data binding disabled directly on an {@code @ModelAttribute}
	 * method parameter or on the attribute returned from an {@code @ModelAttribute}
	 * method, both of which would prevent data binding for that attribute.
	 * <p>By default this is set to {@code true} in which case data binding applies.
	 * Set this to {@code false} to disable data binding.
	 * @since 4.3
	 */
	boolean binding() default true;

}
```

- 왜 사용하는가 ?
    - @RequestParam으로도 충분한 처리가 가능하다.
    - 하지만, 요청 매개변수가 많은 경우라면 ? ..
    - 요청 매개변수가 늘어날수록 Handler Method Argument로 게속해서 늘어날것..
```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@RequestParam Long id, @RequestParam String name, @RequestParam String title, @RequestParam String nickname) {
    Event events = new Event();
    ...
    return events;
}
```

- @ModelAttribute를 사용할 경우 
    - 요청 매개변수 개수의 상관없이 @ModelAttribute를 활용하여 Event 라는 객체로 하나로 받아올수 있다.
    - Event객체를 생성해서, 요청매개변수를 Event 객체로 바인딩 하는 과정의 코드도 사라지게된다.
```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@ModelAttribute Event event) {
    return event;
}
```

- 값을 바인딩 할수 없는경우 ?
    - 400 Error
    - BindingException 발생

- 바인딩 에러를 직접 처리하고 싶은경우
    - BindingResult 를 활용하여 해당 에러를 핸들링이 가능하다.
```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@ModelAttribute Event event, BindingResult result) {
    if (result.hasErrors()) {
        // Error process...
    }
    return event;
}
```

- 바인딩 이후, 검증작업이 필요한 경우
    - @Valid(JSR-303)
    - @Validated(Spring 제공)
    - @Valid 혹은, @Validated 를 사용할 경우, Validation결과에 대한 에러도 BindinResult로 핸들링이 가능하다.
```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@Valid @ModelAttribute Event event, BindingResult result) {
    if (result.hasErrors()) {
        // Error process...
    }
    return event;
}
```
