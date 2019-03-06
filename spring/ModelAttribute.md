### @ModelAttribute
- @RequestParam이 단일타입이라면, ModelAttribute는 컴포짓타입이다.
- 여러개의 요청 매개변수를 하나의 Object형태로 받을 수 있다.
- 해당 객체를 새로 생성할때 사용할 수 있다.
- 타입컨버전을 지원한다.
- @Valid(JSR-303), @Validated(Spring) 을 사용하여 바인딩 후, 검증도 가능하다.
- 바인딩 에러나, 검증에러가 발생한 뒤 핸들러 메서드에서 처리하고싶다면
- 해당 컴포짓 객체 바로 우측에다가 BindingResult를 아규먼트로 받아, 해당 처리 결과를 핸들링 할 수 있다.
- 생략이 가능하다.

```
    @PostMapping("/events/create")
    public String create(
            @Validated @ModelAttribute Event event
            , BindingResult bindingResult
            , SessionStatus status
    ){
        if(bindingResult.hasErrors()){
            return "events/form";
        }

        return "events/list";
    }

```