# Spring 5 in Action

## 7장 REST 서비스 사용하기
- 스프링에서 제공하는 REST API Client
  1. RestTemplate - 스프링 프레임워크에서 제공
  2. Traverson - 스프링 HATEOAS 에서 제공하는 하이퍼링크를 인식하는 클라이언트 자바스크립트 라이브러리를 모티브로 만들어짐
  3. WebClient - 스프링 5에서 소개된 비동기 REST 클라이언트
  

### RestTemplate
- Spring 3.0 부터 지원
- 이름에서 알수 있듯이, JdbcTemplate 처럼 반복적인 코드를 사용하기 편하게 추상화 해둔 클래스
- org.springframework.http.client
- 내부적으로 HttpClient 를 사용하는데, 기본 구현체는 HttpUrlConnection (JDK) 를 사용한다.
- HttpUrlConnection 이 기본 구현체인 이유 ? 
  - -> JDK 에서 기본적으로 제공하고, 단일 성능만 놓고 봤을때는 가장 우수하다. 또한 대용량 트래픽을 다루는 서비스가 아니라면 큰 문제가 없다.
- 4.3 버전부터 OkHttp 를 지원한다.
- 특징은 MessageConverter 를 사용해서 메시지 변환을 처리한다.

`주요 메소드`

| 메소드 | 설명 |
| --- | --- |
| delete() | 지정한 URL 리소스에 HTTP DELETE 요청을 수행한다. |
| exchange() | 지정된 HTTP 메소드를 URL 에 대해 실행하며, 응답 몸체와 연결되는 객체를 포함하는 ResponseEntity 를 반환 |
| execute() | 지정된 HTTP 메소드를 URL 에 대해 실행하며, 응답 몸체와 연결되는 객체를 반환 |
| getForEntity() | HTTP GET 요청을 전송하며, 응답 몸체와 연결되는 객체를 포함하는 ResponseEntity 를 반환 |
| getForObject() | HTTP GET 요청을 전송하며, 응답 몸체와 연결되는 객체를 반환 |
| headForHeaders() | HTTP HEAD 요청을 전송하며, 지정된 리소스 URL 의 HTTP 헤더를 반환 |
| optionsForAllow() | HTTP OPTIONS 요청을 전송하며, 지정된 URL 의 Allow 헤더를 반환 |
| patchForObject() | HTTP PATCH 요청을 전송하며, 응답 몸체와 연결되는 결과 객체를 반환 |
| postForEntity() | URL 에 데이터를 POST 하며, 응답 몸체와 연결되는 객체를 포함하는 ResponseEntity 를 반환 |
| postForLocation() | URL 에 데이터를 POST 하며, 새로 생성된 리소스의 URL 을 반환 |
| postForObject() | URL 에 데이터를 POST 하며, 응답 몸체와 연결되는 객체를 반환 |
| put() | 리소스 데이터를 지정한 URL 에 PUT |


#### RestTemplate 사용하기
- RestTemplate 을 사용하려면 직접 인스턴스를 사용하거나, 빈으로 등록하여 필요시 주입하여 사용할 수 있다.
```java
RestTemplate rest = new RestTemplate();

@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

`GET`
```java
@GetMapping("/ingredient/{id}")
public Ingredient getIngredientById(@PathVariable String id) {
    Ingredient ingredient = restTemplate.getForObject("http://localhost:8080/api/ingredients/{id}", Ingredient.class, id);

    // URI 타입 파라미터를 사용할 경우 URI 객체를 구성해서 사용해야 한다.
    URI uri = UriComponentsBuilder.fromHttpUrl("http://localhost:8080/api/ingredients/{id}")
            .build(Map.of("id", id));
    restTemplate.getForObject(uri, Ingredient.class);

    // getForEntity 는 ResponseEntity 타입을 반환한다. 응답 헤더와 같은 상셍한 내용 조회 가능
    ResponseEntity<Ingredient> responseEntity = restTemplate.getForEntity(uri, Ingredient.class);
    return ingredient;
}
```

`POST`
```java
@PostMapping("/ingredient")
public Ingredient createIngredient(@RequestBody Ingredient ingredient) {
    return restTemplate.postForObject("http://localhost:8080/api/ingredients", ingredient, Ingredient.class);
}
```

`PUT`
```java
@PutMapping("/ingredient/{id}")
public void updateIngredient(@PathVariable String id, @RequestBody Ingredient ingredient) {
    restTemplate.put("http://localhost:8080/api/ingredients/{id}", ingredient, id);
}
```

`DELETE`
```java
@DeleteMapping("/ingredient/{id}")
public void deleteIngredient(@PathVariable String id) {
    restTemplate.delete("http://localhost:8080/api/ingredients/{id}", id);
}
```