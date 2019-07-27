# Spring Handler Method - @ModelAttribute
- @RequestParam과 같이 요청 매개변수를 매핑하는 방법중 하나이다.

- @ModelAttribute
    - 단순 데이터 타입을 하나의 복합타입의 객체로 받아오거나, 객체를 새로 생성할때 사용할수 있다.
    - URLPath, 요청매개변수, 세션 등 ..
    - 생략이 가능하다.

- 왜 사용하는가 ?
    - @RequestParam으로도 충분한 처리가 가능하다.
    - 하지만, 요청 매개변수가 많은 경우라면 ? ..
    - 요청 매개변수가 늘어날수록 Handler Method Argument로 게속해서 늘어날것..
```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@RequestParam Long id, @RequestParam String name, @RequestParam String title, @RequestParam String nickname) {
    Event events = new Event();
    ...
    return events;
}
```

- @ModelAttribute를 사용할 경우 
    - 요청 매개변수 개수의 상관없이 @ModelAttribute를 활용하여 Event 라는 객체로 하나로 받아올수 있다.
    - Event객체를 생성해서, 요청매개변수를 Event 객체로 바인딩 하는 과정의 코드도 사라지게된다.
```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@ModelAttribute Event event) {
    return event;
}
```

- 값을 바인딩 할수 없는경우 ?
    - 400 Error
    - BindingException 발생

- 바인딩 에러를 직접 처리하고 싶은경우
    - BindingResult 를 활용하여 해당 에러를 핸들링이 가능하다.
```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@ModelAttribute Event event, BindingResult result) {
    if (result.hasErrors()) {
        // Error process...
    }
    return event;
}
```

- 바인딩 이후, 검증작업이 필요한 경우
    - @Valid(JSR-303)
    - @Validated(Spring 제공)
    - @Valid 혹은, @Validated 를 사용할 경우, Validation결과에 대한 에러도 BindinResult로 핸들링이 가능하다.
```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@Valid @ModelAttribute Event event, BindingResult result) {
    if (result.hasErrors()) {
        // Error process...
    }
    return event;
}
```
