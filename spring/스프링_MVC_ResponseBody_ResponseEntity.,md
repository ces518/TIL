# Spring - @ResponseBody & ResponseEntity
- @ResponseBody
	- Method LEVEL 에 사용한다.
	- 해당 Method의 리턴 값을 HttpMessageConverter를 사용하여 응답본문으로 보낸다.
	- accpt-header 정보를 참조하여 적절한 MessageConverter를 사용한다.
		- JSON 이라면 Jackson2HttpMessageConverter...
	- @RestController 를 사용할경우, 모든 Method에 @ResponseBody가 있는것과 동일하기 때문에 생략이 가능하다.

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

/**
 * Annotation that indicates a method return value should be bound to the web
 * response body. Supported for annotated handler methods.
 *
 * <p>As of version 4.0 this annotation can also be added on the type level in
 * which case it is inherited and does not need to be added on the method level.
 *
 * @author Arjen Poutsma
 * @since 3.0
 * @see RequestBody
 * @see RestController
 */
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ResponseBody {

}
```

- FileUpload에서 살펴보았던 Handler Method
	- multipart/form-data 로 파일업로드 요청시 해당 file의 이름을 응답 본문으로 보내주는 Handler
```java
@PostMapping("/file")
@ResponseBody
public String fileUpload (MultipartFile file) {
	return file.getOriginalFilename() + "is Uploaded";
}
```

- ResponseEntity
	- 응답 헤더의 상태코드와 본문을 직접 다루고 싶은경우 사용한다.
	- 자주 사용하는 응답의 경우 static factory method 를 제공한다.
	- 이 자체가 응답본문이기 때문에 @ResponseBody 를 한것과 같다.

- FileDonwload에서 살펴보았던 Handler Method
	- fileName을 Uri Path로부터 받아, 파일 리소스를 읽은뒤 해당 파일 Download 응답을 보내는 Handler
	- Header 정보와, 해당 리소스를 응답본문으로 보내준다.
```java
@GetMapping("/file/{fileName}")
public ResponseEntity fileDownload (@PathVariable String fileName) throws IOException {
	Resource resource = resourceLoader.getResource("classpath:" + fileName);
	File file = resource.getFile();

	Tika tika = new Tika();
	String mediaType =  tika.detect(file);
	return ResponseEntity.ok()
			.header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
			.header(HttpHeaders.CONTENT_TYPE, mediaType)
			.header(HttpHeaders.CONTENT_LENGTH, String.valueOf(file.length()))
			.body(resource);
}
```
