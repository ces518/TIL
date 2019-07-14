# Spring MVC WebMvcCongiturer_Formatter

- Formatter
    - 해당 객체를 문자열로 출력 하거나, 어떤 문자열을 객체로 변환하여 받을수 있다.
    - Formatter를 사용하려면 Formatter<T> Interface를 구현해야 한다.
    - Formatter Interface는 Printer<T> 와, Parser<T> 를 합친것이다.

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

package org.springframework.format;

/**
 * Formats objects of type T.
 * A Formatter is both a Printer <i>and</i> a Parser for an object type.
 *
 * @author Keith Donald
 * @since 3.0
 * @param <T> the type of object this Formatter formats
 */
public interface Formatter<T> extends Printer<T>, Parser<T> {

}
```

- Formatter를 등록하는 방법
    - WebMvcConfigurer의 addFormatters(FormatterRegistry) 메서드 정의
    - 해당 Formatter를 빈으로 등록
        - Spring boot 에서만 가능한 방법

```java
package me.june.springbootmvc;

import org.springframework.context.annotation.Configuration;
import org.springframework.format.FormatterRegistry;
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
}
```
