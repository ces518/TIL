# Spring - URI Pattern Mapping
- 요청 매핑
    - @RequestMapping 애노테이션에서 문자열로 요청을 매핑하는 경우를 말한다.

- 식별자로 매핑
    - @RequestMapping은 패턴을 지원한다.
    - ?: 한글자
    - *: 여러글자
    - **: 여러패스

```java
@Controller
public class MvcController {

    @GetMapping("/hello?")
    @GetMapping("/hello*")
    @GetMapping("/hello/**")
    @ResponseBody
    public String hello () {
        return "hello";
    }
}
```

- ClassLevel에 선언한 @RequestMapping과 조합
    - Class에 선언한 URI 패턴뒤에 이어 매핑한다.

```java
@Controller
@RequestMapping("/mvc")
public class MvcController {

    @GetMapping("/hello?")
    @GetMapping("/hello*")
    @GetMapping("/hello/**")
    @ResponseBody
    public String hello () {
        return "hello";
    }
}
```

- 정규표현식
    - /{name: 정규식}

```java
@Controller
@RequestMapping("/mvc")
public class MvcController {

    @GetMapping("{name: [a-z]}")
    @ResponseBody
    public String hello () {
        return "hello";
    }
}
```

- 패턴이 중복되는경우 ?
    - 가장 구체적으로 매핑되는 핸들러가 선택된다.

- URI 확장자 매핑
    - Spring MVC는 지원하지만, Spring Boot는 기본적으로 사용하지않음.
    - 보안이슈 RFD Attack
    - URI 변수, Path 매개변수, URI인코딩을 사용할때 불명확하다.

- SpringMVC
    - @GetMapping("/hello") 로 Mapping을 하면 , "/hello.*" 도 매핑이된다.
    - /hello.json, /hello.xml 과같은 확장자 매핑이 가능하다.
    - 최근에는 Header정보를 보고 판단하는것을 권장한다.
