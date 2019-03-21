### InitBinder
- Class레벨에 바인더,커스텀발리데이터,포메터등을 설정할수있다.
- 값을 설정해주면 , 해당 값과 이름이 동일한 객체에만 바인더를 적용할수있다.
```
    @InitBinder("event")
    public void eventBinder(WebDataBinder webDataBinder) {
        // id 라는 필드를 바인딩 제한.
        webDataBinder.setAllowedFields("id");
        webDataBinder.addValidators(new EventValidator());
    }
```
