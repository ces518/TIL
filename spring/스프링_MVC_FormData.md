# Spring Handler Method - FormData
- Http 요청으로 FormData를 보낼 경우 처리하는 방법

- thymeleaf 를 View로 활용, thymeleaf 파일 생성
    - classpath:resources/template/events/form.html 파일 생성

- GET /mvc/events/form 로 요청을 보내면, eventForm을 View로 Return하는 Handler 코드 작성
```java
@GetMapping("/mvc/events/form")
public String form (Model model) {
    model.addAttribute("event", new Event());
    return "events/form";
}
```

- form.html
    - @{}: URL 표현식
    - ${}: variable 표현식
    - *{}: selection 표현식
    - th:action="@{/mvc/events}": /mvc/events 로 action 값을 지정
    - th:object="${event}" Model에 event라는 객체를 참조하도록 설정
    - th:field="*{id}" th:object에서 참조한 객체의 필드를 매핑
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Event Form</title>
</head>
<body>
<form action="#" th:action="@{/mvc/events}" th:object="${event}">
    <input type="text" th:field="*{id}"/>
    <input type="text" th:field="*{name}"/>
    <button>등록</button>
</form>
</body>
</html>
```

- id 에 1111, name에 test 라고 매핑해서 보낼경우 다음과 같이 정상적으로 응답
```javascript
{"id":1111,"name":"test","starts":null}
```
