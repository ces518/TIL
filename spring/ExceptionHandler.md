### ExceptionHandler 
- 특정 Exception이나 Custom한 Exception에 대한 핸들러를 정의할수있다
- method arguments로 해당 exception, model ... 등등을 받을수있으며
- 해당 Exception 발생시 다양한 에러 핸들링을 커스텀하게 가능함.
- 여러개의 ExceptionHandler존재시, 가장 구체적인 Exception Handler가 동작한다.
- ex) CustomException이 RuntimeException의 상속구조이다.
- CustomException , RuntimeException 핸들러가있을경우 CustomExceptionHandler가 동작하게된다.
- RESTful APi 의 경우엔 ResponseEntity 로 Return하는 방식을 주로사용한다.
```
@ExceptionHandler
public String eventExceptionHanlder(EventException ex, ModelMap modelMap, HttpServletRequest request) {
    modelMap.addAttribute("message","error...");
    return "error";
}
```