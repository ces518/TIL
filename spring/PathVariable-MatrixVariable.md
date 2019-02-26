
## @PathVariable
- 요청 URI의 일부를 핸들러 메서드의 아규먼트로 받는방법
- 타입커버전 지원
- 값이 반드시 있어야함
- Optional 지원

## @MatrixVariable
- 요청 URI패턴에서 키=값 쌍의 데이터를 핸들러 메서드의 아규먼트로 받는방법
- 타입커버전 지원
- 값이 있어야한다
- Optional지원
- 기본적으로 설정이 되어있지않으며, 추가 설정을 해야만 사용할 수 있다.

```
    @GetMapping("/events/{id}")
    @ResponseBody
    public Event getEvents(
            @PathVariable Integer id
            ,@MatrixVariable String name
    ){
        Event event = new Event();
        event.setId(id);
        event.setName(name);
        return event;
    }

    // webconfig 
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false);
        // url에서 세미콜론 제거 false설정
        configurer.setUrlPathHelper(urlPathHelper);
    }
```
