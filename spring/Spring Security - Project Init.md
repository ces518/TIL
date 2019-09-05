# Spring Security - Project Init
#### 프로젝트 생성
- 의존성
	- Spring Boot 2.1.7
	- lombok
	- thymeleaf
	- devtools

#### Index 페이지 만들기
- 프로젝트를 생성하면 classpath:resources 디렉터리 하위에 templates 디렉터리가 존재한다.
	- 만약 없다면 생성해줄것
- 해당 디렉터리 하위에 index.html 파일을 생성한다.
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Welcome Page</title>
</head>
<body>
    <h1 th:text="Hello World!!">Default Message</h1>
</body>
</html>
```

Template Engine으로 Thymeleaf 를 사용한다.
Thymeleaf 를 사용하기위해서는 namespace 선언을 필요로한다.
- Thyleaf 사용을 위한 namespace 선언부
	- :th 에 해당하는 부분은 일종의 alias이다.
	- 원한다면 다른 alias를 지정할 수 있다.
```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

namespace를 선언하고나면 thymeleaf 표현을 사용할 수 있다.
	- th:text 
		- 텍스트를 출력하는 표현식이다.
		- ${} 표현식을 통해 Controller에서 전달받은 model객체를 참조하여 해당 메시지를 출력이 가능하다.
		- 만약 text에 해당하는 값이 없다면 html 태그 내부에 선언된 값이 Default Message로 출력이된다.
			- Thymeleaf의 장점 이다.
			- Controller를 통해 접근하지 않고 html 파일만 띄웠을경우에도 작업이 가능하다.
			- 프론트엔드와의 협업이 용이하다.
```html
<h1 th:text="${message}">Default Message</h1>
```

- Controller 구현
	- / 로 요청을 보내면 index.html 로 View를 리턴해준다.
	- index.html에서는 message 값인 Hello Spring Security 를 출력 할 것이다.
```java
@Controller
public class SampleController {

    @GetMapping("/")
    public String index (Model model) {
        model.addAttribute("message", "Hello Spring Security");
        return "index";
    }
}
```

##### 그외 페이지들..
- index.html 와 마찬가지로 다른 페이지들도 구현한다.
	- admin.html
	- dashboard.html
	- info.html

- Controller는 인증정보를 활용한다.
	- /dashboard, /admin은 인증정보를 필요로하고, 해당 인증정보에서 사용자 이름과 함께 메시지를 조합하여 출력해준다.
	- /, /info는 인증정보를 필요로 하지않는다.
	- 하지만 / 인덱스 페이지는 만약 인증정보가 존재하면 해당 인증정보에서 사용자 이름과 함께 메시지를 조합하여 출력해준다.
```java
@Controller
public class SampleController {

    @GetMapping("/")
    public String index (Model model, Principal principal) {
        if (principal == null) {
            model.addAttribute("message", "Hello Spring Security");
        } else {
            model.addAttribute("message", "Hello Index" + principal.getName());
        }
        return "index";
    }

    @GetMapping("/info")
    public String info (Model model) {
        model.addAttribute("message", "Hello Info");
        return "info";
    }

    @GetMapping("/dashboard")
    public String dashboard (Model model, Principal principal) {
        model.addAttribute("message", "Hello " + principal.getName());
        return "dashboard";
    }

    @GetMapping("/admin")
    public String admin (Model model, Principal principal) {
        model.addAttribute("message", "Hello Admin" + principal.getName());
        return "admin";
    }
}
```

현재는 인증정보가 존재하지 않기때문에 dashboard 와 admin으로 접근한다면 NullPointerException이 발생한다.
