# Spring Handler Method - @Valid, @Validated
- Spring MVC Handler Method Argument에 사용할수 있으며, 바인딩이후 유효성 검사에 사용된다.

- @Valid, @Validated 를 사용하여 유효성검사를 진행하는데 유효한 값이 바인딩 되지않은경우 
    - 해당 BindinError 가 Model에 담긴다.

- Binding Error 발생시 Model에 담기는 정보
    - Event
        - @ModelAttribute로 받아온 객체
    - BindingResult.event
        - Event객체에 대한 BindingError 정보

- PRG Pattern
    - Post > Redirect > Get
    - Post 이후, 브라우저에서 Refresh를 하더라도 폼 서브밋이 발생하지않도록 하는 Pattern


- @Valid
    - JSR303 annotation
    - @Valid 를 사용하면 애노테이션 기반의 유효성 검사를 진행한다.
    - 그룹을 지정할 수 없다.

```java
public class Event {

    @NotNull
    private Long id;

    @NotBlank
    private String name;
}

@PostMapping("/mvc/events")
@ResponseBody
public Event createEvent (@Valid @ModelAttribute Event event, BindingResult result) {
    if (result.hasErrors()) {
        // Error process...
    }
    return event;
}
```

- @Validated
    - Spring MVC에서 제공하는 애노테이션
    - @Valid와 마찬가지로 애노테이션 기반의 유효성 검사를 진행한다.
    - 그룹을 지정할 수 있다.
    - 그룹을 지정한경우, 해당 그룹일때만 유효성 검사를 진행한다.
```java
public class Event {

    interface ValidateCreate {}
    interface ValidateUpdate {}

    @NotNull(groups = ValidateUpdate.class)
    private Long id;
    
    @NotEmpty(groups = { ValidateCreate.class, ValidateUpdate.class })
    private String name;
}

@PostMapping("/mvc/events")
@ResponseBody
public Event createEvent (@Validated(Event.ValidateCreate.class) @ModelAttribute Event event, BindingResult result) {
    if (result.hasErrors()) {
        // Error process...
    }
    return event;
}
```
