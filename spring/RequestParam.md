### 요청 매개변수
- @RequestParam
- queryString 또는 form data를 요청 매개변수라고한다.
- 요청 매개변수에 있는 단순 타입 데이터를 메서드의 아규먼트로 받아올 수 있다.
- 값이 반드시 있어야한다.
- required=false 이며 이는 Optional로 대체가능하다.
- Optional을 지원한다.
- String이 아닌 데이터타입은 타입 컨버전을 지원한다.
- 생략이가능하다.
- Map<String,String> , MultiValueMap<String,String> 을 사용하여 모든 매개변수들을 받아올 수 있다.
```
    @PostMapping("/events")
    @ResponseBody
    public Event createEvents(
            @RequestParam String name
    ){
        Event event = new Event();
        event.setName(name);
        return event;
    }

    @GetMapping("/events/form")
    public String eventsForm(
            Event event
            ,Model model
    ){
        return "events/form";
    }
```