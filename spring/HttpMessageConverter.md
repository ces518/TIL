# Spring MVC - HttpMessageConverter
- HttpMessageConverter란 ?
    - 요청 본문에서 메시지를 읽어들이거나 , 응답본문에 메시지를 작성할 경우 사용한다.
    - @RequestBody
        - 요청 본문에 있는 메시지를 HttpMessageConverter를 사용해서 Conversion 한다.
    - @ResponseBody
        - 핸들러 메시지의 리턴값 (응답) 을 응답의 본문으로 넣어준다.

- 다음과 같이 @RequestBody를 사용해서 요청 본문에 있는 메시지를 읽어오고,
- @ResponseBody를 사용해서 응답을 본문으로 넣어줄수 있다.
    - @RestController 를 사용하면, 해당 클래스에 존재하는 모든 핸들러메서드는
    - 모든 응답을 본문으로 넣어준다. 
    - @ResponseBody가 생략이 가능함.
    - @ResponseEntity는 응답 자체가 본문이기 때문에 @ReponseBody, @RestController 애노테이션을 사용하지않아도 본문으로 응답하게 된다.

```java
@GetMapping("/message")
@ResponseBody
public String message (@RequestBody String message) {
    return "hello " + message;
}
```

- HttpMessageConverter를 사용하면 가장 기본적인 형태인 문자열 형태로 받거나 응답하고, 객체로도 Conversion이 가능하다.
- 응답 Header의 ContentType에 따라 어떤 MessageConverter를 사용할지 결정이 된다.

- 설정 방법
    - 기본으로 등록해주는 컨버터에 새로운 컨버터를 추가 
        - extendMessageConverters
    - 기본 설정을 무시하고, 새롭게 컨버터를 설정 
        - cofigureMessageConverters
    - 의존성을 추가하여 컨버터 등록
        - Maven or Gradle 에 의존성을 추가하면 그에 따른 컨버터가 자동으로 등ㄹㅗㄱ된다.
        - WebMvcConfigureSupport
        - SpringFramework의 기능이다.

- Basic Http MessageConverter
    - 바이트배열
    - 문자열
    - Resource
        - 옥텟타입의 요청이나 응답을 사용할경우 Resource를 찾아서 응답
    - Form
        - MultiValueMap<String, String>
        - HTML FormData를 Map으로 읽어오거나, 응답이 가능하다.
    - 해당 의존성이 존재할 경우에만 사용한 컨버터들 
        - * JAXB2
            - XML 
        - * Jackson2
            - JSON
        - * Jackson
            - JSON
        - * Gson
            - JSON
        - * Atom
            - Atom
        - * Rss
            - RSS 피드
        - ....


### HttpMessageConverter
- SpringBoot 사용시 
- Jackson이 자동적으로 의존성에 추가되어있기때문에 ObjectMapper가 빈으로 등록된다.
- 기본적인 자동설정으로 HttpMessageConverter가 ObjectMapper를 사용한다.
- HttpMessageConverter Config
- 스프링 Mvc설정에서 설정 할 수 있다.
- configureMessageConverters > 기본 메시지컨버터 대체
- extendMessageConverters > 메시지컨버터에 추가

#### 기본 컨버터
- WebMvcConfigurationSupport.addDefaultHttpMessageConverters

#### HttpEntity
- RequestBody와 비슷하지만 추가적으로 요청헤더정보를 사용 할 수 있다.

#### ResponseBody
- Method LEVEL에 사용가능한 어노테이션이며 , 해당 어노테이션 사용시 HttpMessageConverter를 사용하여  응답본문으로 응답을 보내게된다.
- RestController 어노테이션이 Class LEVEL에 존재한다면, 해당 Class의 모든 메서드에 @ResponseBody가 자동적으로 붙게된다.

#### ResponseEntity
- Method의 Return Type으로 사용이가능하며 , 응답 헤더, 메시지 , 본문, 응답코드 등을 함께 보낼수있다.
-  ResponseEntity가 Return Type이라면 , 그자체가 응답본문이므로 RestController , ResponseBody가 불필요함.
- 다양한 static Method들을 제공하므로 RestAPI 설계시 유용하다.

```
@PostMapping
public Event createEvent(
        @RequestBody @Valid Event event
        , BindingResult bindingResult
) {
    //save
    if(bindingResult.hasErrors()) {
        bindingResult.getAllErrors().forEach(objectError -> System.out.println(objectError));
    }
    return event;
} 
```   
