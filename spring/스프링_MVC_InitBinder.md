# Spring - DataBinder: @InitBinder
- Data를 바인딩할때 사용되는 DATABinder를 커스터마이징이 가능하다.

- @InitBinder
	- Spring 2.5 부터 지원
	- @InitBinder(키)
		- 특정 컨트롤러에서 바인딩 혹은 검증 설정을 커스터마이징할때 사용한다.
	- 리턴값은 반드시 void 여야한다.
	- Method LEVEL에 선언한다.
	- WebDataBinder 객체를 Argument로 받아 커스터마이징을 할수 있다.

```java
/*
 * Copyright 2002-2012 the original author or authors.
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

/**
 * Annotation that identifies methods which initialize the
 * {@link org.springframework.web.bind.WebDataBinder} which
 * will be used for populating command and form object arguments
 * of annotated handler methods.
 *
 * <p>Such init-binder methods support all arguments that {@link RequestMapping}
 * supports, except for command/form objects and corresponding validation result
 * objects. Init-binder methods must not have a return value; they are usually
 * declared as {@code void}.
 *
 * <p>Typical arguments are {@link org.springframework.web.bind.WebDataBinder}
 * in combination with {@link org.springframework.web.context.request.WebRequest}
 * or {@link java.util.Locale}, allowing to register context-specific editors.
 *
 * @author Juergen Hoeller
 * @since 2.5
 * @see org.springframework.web.bind.WebDataBinder
 * @see org.springframework.web.context.request.WebRequest
 */
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface InitBinder {

	/**
	 * The names of command/form attributes and/or request parameters
	 * that this init-binder method is supposed to apply to.
	 * <p>Default is to apply to all command/form attributes and all request parameters
	 * processed by the annotated handler class. Specifying model attribute names or
	 * request parameter names here restricts the init-binder method to those specific
	 * attributes/parameters, with different init-binder methods typically applying to
	 * different groups of attributes or parameters.
	 */
	String[] value() default {};

}
```

```java
@InitBinder
public void initEventBinder (WebDataBinder webDataBinder) {
	...
}
```

- Binding 관련 설정
	- setDisallowedFields("필드명"); 바인딩을 받고싶지 않은 필드를 설정

```java
@InitBinder
public void initEventBinder (WebDataBinder webDataBinder) {
	webDataBinder.setDisallowedFields("id");
}
```

- Formatter 관련 설정
	- webDataBinder.addCustomFormatter(): Spring이 지원하지않는 Formatter를 등록
	- @DateTimeFormat(iso = DateTimeFormat.DATE) 을 사용할수 있는 이유 ?
		- 이를 이해하는 Formatter가 등록되어 있기 때문이다.
		- web과 관련된부분은 formatter가 특화되어있음.
```java
@InitBinder
public void initEventBinder (WebDataBinder webDataBinder) {
	webDataBinder.addCustomFormatter(new MyFormatter());
}
```

- Validator 관련 설정
	- webDataBinder.addValidators();
```java
@InitBinder
public void initEventBinder (WebDataBinder webDataBinder) {
	webDataBinder.addValidators(new MyValidator());
}
```
