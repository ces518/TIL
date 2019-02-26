
## Http Method 를 이용한 매핑.
- HEAD = get 요청으로 보내지만, body에 본문을 보내지않고, 헤더정보만 받아온다.
    (해당 리소스가 요청을 잘 처리하고 응답을 하는지 확인하는 의미)
- OPTIONS = 해당 리소스가 어떠한 Method들을 지원하는지 받아온다.
> 위 기능들은 Spring Mvc를 사용한다면 기본적으로 구현이되어있다.
```
    @GetMapping("/head")
    @ResponseBody
    public String head(){
        return "head";
    }

    @PostMapping("/head")
    @ResponseBody
    public String heads(){
        return "heads";
    }
```