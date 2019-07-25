# Spring Handler Method - URI Pattern
- 요청 URI 패턴의 일부를 Method Arguement로 받을 수 있다.

- @PathVariable
    - 요청 URI 패턴의 일부를 핸들러 메서드의 Argument로 받아올 수 있다.
    - 타입변환 지원
    - java1.8 부터 Optional을 지원한다. (requried false와 동일)
    - required 속성을 통해 필수 값 유무를 지정가능 기본은 true

- Event Handler 
```java
@GetMapping("/events/{id}")
@ResponseBody
public Event getEvent (@PathVariable Long id) {
    Event event = new Event();
    event.setId(id);
    return event;
}
```

- @MatrixVaraible
    - RFC 3985
    - 요청 URI 패턴에서 key/value 쌍의 데이터를 Method Argument로 받을수 있다.
    - 타입변환 지원
    - 값이 반드시 있어야한다.
    - Optional을 지원한다.
    - 이 기능을 기본적으로 비활성화 되어있으며, 활성화시 추가적인 설정이 필요하다.

- WebConfig.java
    - 기본적으로 UrlPathHelper 가 ; 세미콜론을 지워버리기 때문에 MatrixVariable을 매핑할 수 없음
    - 따라서 세미콜론을 지우는옵션을 false로 변경해서 새로이 등록해준다.
```java
@Override
public void configurePathMatch(PathMatchConfigurer configurer) {
    UrlPathHelper urlPathHelper = new UrlPathHelper();
    urlPathHelper.setRemoveSemicolonContent(false);
    configurer.setUrlPathHelper(urlPathHelper);
}
```

- Event Handler 
    - GET http://localhost:8080/events/15;q=11/pets/42;/q=22
    - 위의 매핑의 경우를 다음과 같이 MatrixVariable로 받아올 수 있다.
    - 15;q=11 부분이 모두 ownerId에 해당하는 파트
    - 42;/q=22 부분이 모두 petId에 해당하는 파트
    - 해당 파트중 키,벨류쌍읜 부분을 MatrixVariable로 받아올 수있음.
```java
@GetMapping("/events/{ownerId}/pets/{petId}")
@ResponseBody
public Event getEvent (@MatrixVariable(name="a", pathVar="ownerId") int q1, 
                        @MatrixVariable(name="a", pathVar="petId") int q2) {
    Event event = new Event();
    event.setId(id);
    return event;
}
```
- Map으로 받아오는 방법
    - GET http://localhost:8080/events/15;q=11/pets/42;/q=22
    - MultiValueMap 을 활용해서 받아올 수 있으며, pathVar를 선언하지않으면 모든 MatrixVariable에 해당하는값을 한번에 받아온다.
    - {"q": [11, 22]}
    - @MatrixVariable(pathVar="petId") 와 같이 pathVar를 지정해주면, 해당부분의 MatrixVariable만 가져올 수 있다.
    - {"q": 22}
```java
@GetMapping("/events/{ownerId}/pets/{petId}")
@ResponseBody
public Event getEvent (@MatrixVariable MultiValueMap<String, String> matrixVars, 
                        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {
    Event event = new Event();
    event.setId(id);
    return event;
}
```
