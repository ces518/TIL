### RedirectAttribute
- Spring Web Mvc 는 redirect 수행시 , 기본적으로 Model에 primitive type의 데이터들을
- 자동으로 queryString으로 붙여준다. (Spring boot 는 기본으로 off )
- Spring Boot
> RequestMappingHandlerAdapter 설정에 setIgnoreDefaultModelOnRedirect = true 로 되어있음.
- true일경우 자동으로 붙지않는다.

- redirectAttributes의 addAttribute
- model에 있는 값들을 모두 붙이지않고,
- 원하는 특정값만 redirect하기를 원한다면 , redirectAttributes의 addAttribute로 추가해주면 해당값들만 queryString으로 붙게된다.
- > String으로 변환이 가능한 데이터만 가능하다. primitive type

- redirectAttributes의 flashAttribute
- 말그대로 1회성 이며 , session에 담겼다가 redirect가 완료된 후 사라진다.
- @ModelAttribute 로 받을수 있으며,
- 핸들러 메서드의 아규먼트로  Model이 선언되어있다면, Model에 자동적으로 들어오기때문에
- 꺼내서 사용이가능하다.
```
    @GetMapping("/redirectAttributes")
    public String redirectAttributes(
            RedirectAttributes redirectAttributes
    ){

        Event newEvent = new Event();
        newEvent.setName("june");
        newEvent.setId(2);

        redirectAttributes.addAttribute("name","june");
        redirectAttributes.addFlashAttribute("newEvent",newEvent);
        return "redirect:/test";
    }
```