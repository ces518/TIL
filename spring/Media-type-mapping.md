## MediaType Mapping 

- RequestMapping 의 attributes 중..
- 1. consumes에 contentsType을 명시해주면 , 해당 contentType의 요청만 받아 오도록 매핑할 수 있다.
org.springframework.http.MediaType.class의 상수를 사용하는것이좋다.
- **_VALUE는 문자열을 리턴, ** 는 MediaType을 리턴한다.
- 매치되지 않는경우 415 error code (Not Supported Media Type) 

- 2. produces 에 contentType을 명시해주면 해당 contentType의 응답만 처리하도록 매핑할 수 있다. (accept Header정보를 참조함)
- 약간 오묘한 점이있다... accept 헤더에 아무런 정보가 없다면 , 아무런 타입으로 응답을 받겠다는 의미이기 때문에.. 매핑이된다 .... ?! 
- * ClassLEVEL에도 설정이 가능한데.. ClassLEVEL에 설정한 정보와 조합되지않고, MethodLEVEL에 설정한 정보로 overwrite 된다.
```
    @RequestMapping(value = "/media"
            ,consumes = MediaType.APPLICATION_JSON_UTF8_VALUE
            ,produces = MediaType.TEXT_PLAIN_VALUE
                    )
    @ResponseBody
    public String media(){
        return "json";
    }
```