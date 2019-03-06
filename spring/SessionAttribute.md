### @SessionAttribute
- SessionAttribute와 ModelAttribute가 class LEVEL에서의 관리라면
- SessionAttribute는 Application LEVEL에서의 관리이다.
- Filter나 Interceptor등에서 세션에 담아준 정보를 가져올때 유용하다.
- 타입 컨버전을 지원한다.
- 해당하는 attribute가 존재하지않는다면, 400 Bad Request가 발생한다.
```
  @GetMapping("/session")
    @ResponseBody
    public EventDTO session(@SessionAttribute EventDTO eventDTO) {
        System.out.printf("eventDTO = %s",eventDTO.getName());
        return eventDTO;
    }
```
