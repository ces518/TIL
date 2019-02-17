## URI Pattern Mapping 
- /uri?  : uri 뒤에 1글자가 와야한다.
- /uri*  : uri 뒤에 여러글자가 올수있다.
- /uri/* : uri 뒤에 1개의 path만 매핑한다.
- /uri/**: uri 뒤에 여러개의 path매핑
- {name: 정규식} : pathvariable로 받아올 문자를 정규식 체크가 가능하다.

- 다음과 같이 uri 매핑정보가 중복되는경우 , 1번과 같이 명확한 매핑을 가진 핸들러가 우선순위를 가진다.
- 1./june
- 2./**

- URI 확장자 패턴 지원 (SpringBoot는 기본적으로 off)
- RDF Attack (Reflected File Download) > Spring4.3 이전버전에는 해당 이슈가 존재함.
- /june 으로 매핑하면 /june.* 의 확장자 패턴 매핑을 자동으로 설정해준다.
- ps . 확장자패턴은 최근에는 걷어내는 추세이다.. 다양한 타입의 데이터를 제공해야한다면 , contentType에 싣는것을 가장 먼저 고려해볼것.
- 차선책으로는 파라메터에 싣는것을 권장한다.
```
    @GetMapping("/uri/{name:[a-z]}")
    @ResponseBody
    public String helloUri(@PathVariable String name){
        return "hello " + name;
    }
```