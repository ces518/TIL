# Spring Handler Method - @RedirectAttributes
- Redirect시 Model에 존재하는 Primitive Type데이터는 자동으로 URI QueryParameter로 추가된다.
    - 스프링부트 에서는 이 기능이 기본적으로 비활성화 되어있다.
    - RequestMappingHandlerAdapter 설정에 setIgnoreDefaultModelOnRedirect = true 로 되어있음.
    - Ignore-default-model-on-redirect 옵션을 사용해서 활성화 할 수 있음.
    - Model에 존재하는 모든 Primitive Type DATA들을 추가하지않고, 원하는 데이터만 명시적으로 추가하고 싶다면 RedirectAttributes를 사용한다.

- RedirectAttributes   
    - Spring 3.1 이상 제공 
    - Redirect시 원하는 값들을 전달하고 싶다면 RedirectAttributes를 사용해서 추가할 수 있다.
    - addAttribute로 담긴 primitive type data는 QueryParameter로 전달이된다.
    - URI 로 전달되기 때문에 문자열로 변환이 가능해야한다.
```java
@PostMapping("/mvc/events")
public String hello (@ModelAttribute Event event, BindingResult result, RedirectAttributes attributes) {
    if (result.hasErrors()) {
        // Error process...
    }
    attributes.addAttribute("name", event.getName());
    return "redirect:/mvc/events";
}
```

#### 주의점
- RedirectAttributes를 사용할때, @SessionAttributes와 함께 사용한다면 주의해서 개발을 해야한다.
- 1. RedirectAttributes 사용시 @SessionAttributes에 지정해둔 name값과 동일한 상태
- 2. sessionStatus.setComplete() 를 활용하여 세션을 비워진 상태
- 3. @ModelAttribute 로 복합 객체를 받아올때 @SessionAttributes에 설정해둔 name값에 해당하는 객체를 session에서 먼저 찾으려고 하고, 없다면 예외가 발생하게된다.

- Flash Attributes
    - 주로 리다이렉트시 데이터를 전달할때 사용된다.
    - DATA가 URI에 노출되지않는다.
    - 임의의 객체를 저장할 수 있다.
    - HTTP Session을 활용한다.
    - Redirect전 DATA를 HTTP Session에 저장하고, Redirect후 즉시 제거한다.
    - Model을 선언해두었다면 FlahsAttributes로 넘긴 데이터가 model에 담겨있기때문에, Model만 선언해두어도 model에서 꺼내어 사용할 수 있다.
    - @ModelAttribute나 @RequestParam등으로 받으려고 선언하지않아도 된다.

```java
@PostMapping("/mvc/events")
public String hello (@ModelAttribute Event event, BindingResult result, RedirectAttributes attributes) {
    if (result.hasErrors()) {
        // Error process...
    }
    attributes.addFlashAttribute("event", event);
    return "redirect:/mvc/events";
}
```

#### 정리
- Spring MVC는 Redirect시 Model에 존재하는 모든 Primitive Type DATA를 URI QueryParameter로 추가한다.
- Spring Boot는 기본적으로 이 설정이 OFF
    - Ignore-default-model-on-redirect 옵션으로 ON/OFF 가능
- Redirect시 원한는 값들만 명시적으로 추가하고 싶다면 RedirectAttributes를 사용할것.
    - addAttribute로 담긴 Primitive Type DATA는 URI QueryParameter로 전달이된다.
    - URI 로 전달되기 때문에, 문자열로 변환이 가능해야한다.
- @SessionAttributes, @ModelAttribute와 함께 사용시 name값에 각별히 주의할것.
- Redirect시 URI 에 파라미터로 노출되고싶지않은 데이터가 있다면 FlashAttributes를 사용할것.
    - DATA가 URI에 노출되지않으며, Object도 전달이 가능하다.
    - Session에 저장되었다가 Redirect후 즉시 제거된다.
    - FlahsAttributes로 전달한 데이터는 Model에 담기기 때문에 Model만 선언해두었디만, @ModelAttribute나 @RequestParam으로 받지 않아도 된다.

