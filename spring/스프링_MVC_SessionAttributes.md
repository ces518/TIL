# Spring Handler Method - @SessionAttributes
- Model정보를 HTTP Session에 저장해주는 애노테이션

- @SessionAttributes
    - HttpSession을 직접 사용할수도 있지만 애노테이션에 설정한 name, type 에 해당하는 모델정보를 자동으로 sessions에 넣어준다.
    - @ModelAttribute는 Session에 존재하는 데이터도 바인딩한다.
    - 여러화면 (요청)에서 사용해야하는 객체를 공유할때 사용한다.
    - Class Level에 선언해야한다.
```java
@Controller
@SessionAttributes("event")
public class MvcController {

    @GetMapping("/mvc/events/form")
    public String form (Model model) {
        model.addAttribute("event", new Event());
        return "events/form";
    }

    @GetMapping("/mvc/events")
    @ResponseBody
    public Event hello (@ModelAttribute Event event, BindingResult result, HttpSession session) {
        Event sessionedEvent = (Event) session.getAttribute("event");
        System.out.print(event == sessionedEvent); // true
        if (result.hasErrors()) {
            // Error process...
        }
        return event;
    }
}
```

- SessionStatus
    - @SessionAttributes를 사용해서 저장된 객체를 세션에서 비워줄때 사용한다.
    - sessionStatus.setComplete()

```java
@GetMapping("/mvc/events")
@ResponseBody
public Event hello (@ModelAttribute Event event, BindingResult result, SessionStatus status) {
    if (result.hasErrors()) {
        // Error process...
    }
    status.setComplete();
    return event;
}
```
