### SessionAttributes
- type or name 으로 지정하여, 해당 name(type)에 해당하는 값이 model에 들어올경우
- Session에 담아준다.
- 해당 모델을 세션에서 지워야할 경우, sessionStatus.setComplete(); 메서드로알려주어야한다.
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

        status.setComplete();

        return "events/list";
    }
```