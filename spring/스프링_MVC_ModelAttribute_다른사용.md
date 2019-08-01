# Spring - @ModelAttribute의 다른 사용법
- @ModelAttribute
	- @RequestMapping을 사용한 Handler Method Argument에 사용
	- @Controller or @ControllerAdvice를 사용한 클래스에서 Model 정보를 초기화할때 사용한다.
		- 공통적으로 참조해야하는 정보가 있는경우
	- @RequestMapping과 함께 사용하면 해당 Method에서 Return하는 객체를 Model에 담아준다.
		- RequestToViewNameTranslator 를 사용 요청으로부터 ViewName을 유추하여 View를 Return해준다.

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

- Handler Method에서 공통적으로 참조하는 Model 정보를 만드는 2가지방법
	- 1. model객체에 직접 담아준다.
	- 2. @ModelAttribute("키값") 형태로 정의해준다면 return하는 객체를 해당하는 키값으로 담아준다.
		- 키값을 지정하지않는경우 임의의 키값으로 담아준다 (기본전략은 해당 Class명 camelCase)
```java
@ModelAttribute
public void categories (Model model) {
	model.addAttribute("categories", Arrays.asList("study", "hobby"));
}

@ModelAttribute("categories")
public List<String> categories () {
	return Arrays.asList("study", "hobby");
}
```
