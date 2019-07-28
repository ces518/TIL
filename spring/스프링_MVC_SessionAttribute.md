# Spring Handler Method - @SessionAttribute
- HTTP 세션값에 들어있는 값을 참조할때 사용한다.

- @SessionAttribute
    - HttpSession을 사용할때 비해, 타입 컨버전을 자동으로 지원하기때문에 편리하다.
    - HTTP 세션에 데이터를 넣고, 빼고 싶은경우에는 HttpSession 을 사용.

```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@ModelAttribute Event event, BindingResult result, @SessionAttribute LocalDateTime visitTime) {
    if (result.hasErrors()) {
        // Error process...
    }
    return event;
}
```

- @SessionAttributes와 차이점
    - @SessionAttributes는 해당 컨트롤러 내에서만 동작한다.
        - 해당 컨트롤러 안에서 다루는 특정 객체를 세션에 넣고 공유할때 사용
    - @SessionAttribute는 컨트롤러 밖 (Interceptor, Filter 등) 에서 만들어준 값을 참조할때 사용한다.
