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