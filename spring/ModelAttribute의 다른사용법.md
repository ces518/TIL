### ModelAttribute 의 또 다른 사용법
- @Controller , @ControllerAdvice 에서 사용할 클래스에서 모델정보를 초기화할때 사용한다.
- event라는 이름의 모델이 없을경우 , 초기화로 사용되며, 존재하는경우에는 동작하지않는다.
- @RequestMapping과 함께사용할경우 해당 핸들러 메소드에서 리턴하는 객체를 Model에 넣어준다.
```
@ModelAttribute("event")
    public Event event() {
        Event event = new Event();
        event.setName("june0");
        return event;
    }
```