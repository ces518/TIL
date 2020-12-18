# Spring 5 in Action

## 2장 웹 애플리케이션 개발하기

### 정보 보여주기
- 타코 클라우드라는 샘플 애플리케이션을 만들것이다.
- 타코 클라우드는 온라인으로 타코를 주문 할 수 있는 애플리케이션 이다.
- 또한 풍분한 식자제 (ingredient) 를 보여주는팔레터를 사용해서 커스텀한 타코를 디자인 할 수 있도록 한다.
- 원하는 식자재를 보여주는 HTML 페이지가 있어야 하고, 선택가능한 식자재 내역은 수시로 변경될 수 있다.
- HTML 에 하드코딩 되어서는 안되고, 데이터베이스로 부터 고나리 되어야 ㅎ나다.
- 애플리케이션에서 데이터를 가져오고 처리하는것이 컨트롤러의 역할이다.
- 크게 3가지 컴포넌트를 생성할 것이다.
    1. 타코 식자재를 정의하는 도메인 클래스
    2. 식자재 정보를 가져와 뷰에 전달하는 MVC 컨트롤러 클래스
    3. 식자재의 내역을 사용자 브라우저에 보여주는 템플릿

### 도메인 구성
- 애플리케이션의 도메인은 해당 애플리케이션의 이해에 필요한 개념을 다루는 영역이다.
- 도메인에 필요한 객체는 다음과 같다.
    1. 타코 디자인
    2. 디자인을 구성하는 식자재
    3. 고객
    4. 고객의 타코 주문

#### 타코 식자재 정의
```java
@Data
class Ingredient {
    private final String id;
    private final String name;
    private final Type type;
    
    public enum Type {
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }
}
```
- **Lombok** 라이브러리를 사용하면 getter, setter 등 보일러플레이트 코드를 줄여준다.
    - https://projectlombok.org
- @Data 애노테이션을 사용할 때는 주의해야 한다.
    - 같은 타입의 필드 순서가 변경되면 이를 감지하지 못하여 잘못된 초기화가 될 여지가 있다.
    - 무분별한 setter 사용이 될 여지가 있다.
    - https://kwonnam.pe.kr/wiki/java/lombok/pitfall
    - https://www.popit.kr/실무에서-lombok-사용법/
    
#### 타코 디자인 정의
```java
class Taco {
    private String name;
    private List<String> ingredients;
}
```

### 컨트롤러 구성
- 타코 클라우드 애플리케이션에 필요한 컨트롤러는 다음과 같다.
    1. HTTP GET /design 요청을 처리
    2. 식자재 내역 생성
    3. 식자재 데이터를 HTML 로 웹브라우저로 응답

```java
@Slf4j
@Controller
@RequestMapping("/design")
public class DesignTacoController {

    @GetMapping
    public String showDesignForm(Model model) {
        List<Ingredient> ingredients = List.of(
                new Ingredient("FLTO", "Flour Tortilla", Ingredient.Type.WRAP),
                new Ingredient("COTO", "Corn Tortilla", Ingredient.Type.WRAP),
                new Ingredient("GRBF", "Ground Beef", Ingredient.Type.PROTEIN),
                new Ingredient("CARN", "Carnitas", Ingredient.Type.PROTEIN),
                new Ingredient("TMTO", "Diced Tomatoes", Ingredient.Type.VEGGIES),
                new Ingredient("LETC", "Lettuce", Ingredient.Type.VEGGIES),
                new Ingredient("CHED", "Cheddar", Ingredient.Type.CHEESE),
                new Ingredient("JACK", "Monterrey Jack", Ingredient.Type.CHEESE),
                new Ingredient("SLSA", "Salsa", Ingredient.Type.SAUCE),
                new Ingredient("SRCR", "Sour Cream", Ingredient.Type.SAUCE)
        );

        for (Ingredient.Type type : Ingredient.Type.values()) {
            model.addAttribute(type.toString().toLowerCase(), ingredients.stream().filter(x -> x.getType() == type).collect(Collectors.toList()));
        }

        model.addAttribute("taco", new Taco());

        return "design";
    }

    @PostMapping
    public String processDesign(Taco design) {
        // 저장 작업은 3장에서 ..
        log.info("Processing design: " + design);

        return "redirect:/orders/current";
    }
}
```
- @Slf4j 애노테이션은 SLF4J Logger 를 생성한다.
- @Controller 는 DesignTacoController 를 컨트롤러로 식별하게끔 한다.
- @Controller 애노테이션의 선언을 살펴보면, 특별한 기능이 추가된것은 아니고, @Component 의 메타애노테이션이다.
- 네이밍을 통해 좀 더 의미를 전달하기 위해 사용한다. 실질적인 동작은 @Component 를 사용하더라도 동일하게 동작한다.
- 위 processDesign 메소드는 입력에 대한 데이터 처리를 수행한후 redirect 를 하는데 이런 패턴을 **PRG 패턴 (Post-Redirect-Get)** 이라고 한다.
- https://it-eldorado.tistory.com/68
  
`@Controller 선언`
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```

- @RequestMapping 해당 컨트롤러가 핸들링할 URL 매핑정보를 제공하는 애노테이션이다.
- @RequestMapping 은 선언가능한 곳이 두 가지이다.
- 하나는 클래스레벨, 하나는 메소드 레벨이다.
- 클래스 레벨에 선언될 경우 해당 클래스 내부에 존재하는 메소드의 매핑에 prefix 형태로 처리된다.
    - Class Level 에 /taco 로 매핑을 해두었다면 하위 메소드의 매핑 앞에 모두 /taco 가 붙게된다.
- 특정 HTTP Method 만 처리하게끔 선언하거나, 특정 Content-Type 요청만 처리 등 다양한 설정이 가능하다. 

`@RequestMapping 선언`    
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    String name() default "";

    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};

    String[] params() default {};

    String[] headers() default {};

    String[] consumes() default {};

    String[] produces() default {};
}
```

- @RequestMapping 을 좀 더 확장해서, 특정 HTTP Method 에 특화된 애노테이션도 존재한다.

| 애노테이션 | 설명 |
| --- | --- |
| @GetMapping | GET 요청 처리 |
| @PostMapping | POST 요청 처리 |
| @PutMapping | PUT 요청 처리 |
| @DeleteMapping | DELETE 요청 처리 |
| @PatchMapping | PATCH 요청 처리 |

> 위의 애노테이션들은 @RequestMapping 의 method 속성을 정의한것과 동일하다.
> 좀 더 명시적이고 가독성이 좋기때문에 스프링 4.3 버전 이상을 사용하고 있다면, 위 애노테이션들을 사용할것을 권장한다.

### 뷰 구성
- 뷰 템플릿 라이브러리로 Thymeleaf 를 사용한다.
- Thymeleaf 는 스프링에서 공식으로 지원하는 템플릿 라이브러리이다.
- 다음은 타임리프를 사용하여 구성한 design.html 이다

`design.html`
```html
<!DOCTYPE html>
<html lang="ko"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Taco Cloud</title>
    <link ref="stylesheet" th:href="@{/style.css}"/>
</head>
<body>
    <h1>Design your taco!</h1>
    <img th:src="@{/images/TacoCloud.png}"/>

    <form method="POST", th:object="${taco}">
        <span class="validationError"
            th:if="${#fields.hasErrors('ingredients')}"
            th:error="*{ingredients}">Ingredient Error</span>

        <div class="grid">
            <div class="ingredient-group" id="wraps">
                <h3>Designate your wrap:</h3>

                <div th:each="ingredient : ${wrap}">
                    <input name="ingredients" type="checkbox" th:value="${ingredient.id}"/>
                    <span th:text="${ingredient.name}">INGREDIENT</span><br/>
                </div>
            </div>

            <div class="ingredient-group" id="proteins">
                <h3>Pick your protein:</h3>

                <div th:each="ingredient : ${protein}">
                    <input name="ingredients" type="checkbox" th:value="${ingredient.id}"/>
                    <span th:text="${ingredient.name}">INGREDIENT</span><br/>
                </div>
            </div>

            <div class="ingredient-group" id="cheeses">
                <h3>Choose your cheese:</h3>

                <div th:each="ingredient : ${cheese}">
                    <input name="ingredients" type="checkbox" th:value="${ingredient.id}"/>
                    <span th:text="${ingredient.name}">INGREDIENT</span><br/>
                </div>
            </div>

            <div class="ingredient-group" id="veggies">
                <h3>Determine your veggies:</h3>

                <div th:each="ingredient : ${veggies}">
                    <input name="ingredients" type="checkbox" th:value="${ingredient.id}"/>
                    <span th:text="${ingredient.name}">INGREDIENT</span><br/>
                </div>
            </div>

            <div class="ingredient-group" id="sauces">
                <h3>Select your sauce:</h3>

                <div th:each="ingredient : ${sauce}">
                    <input name="ingredients" type="checkbox" th:value="${ingredient.id}"/>
                    <span th:text="${ingredient.name}">INGREDIENT</span><br/>
                </div>
            </div>
        </div>

        <div>
            <h3>Name your taco creation: </h3>
            <input type="text" th:field="*{name}"/>
            <span th:text="${#fields.hasErrors('name')}">XXX</span>
            <span class="validationError"
                th:if="${#fields.hasErrors('name')}"
                th:error="*{name}">Name Error</span>
            <br/>

            <button>Submit your taco</button>
        </div>
    </form>

</body>
</html>
```

- xmlns:th 를 사용해 타임리프 namespace 를 사용 가능하도록 선언해 준다.
- th: 로 시작하는 속성들은 모두 타임리프 문법이다.
- 자세한 설명은 아래 링크를 참조
- https://www.thymeleaf.org/documentation.html

### 폼 제출 처리
- 위 html 을 살펴보면 method = POST 로 선언되어 있다.
- 또한 action 속성이 정의되어 있지 않기 때문에 해당 form 이 submit 되면, 현재 URL 로 제출이 된다. (/design)
- 위에서 정의한 processDesign 메소드가 이를 처리하게 된다.

### 유효성 검사 처리
- 각 폼이 제출될 때 마다 유효성검사를 할 필요가 있다.
- 스프링은 자바의 빈 유효성 검사 Bean Validation API (JSR-303) 을 지원한다.
    - 구현체로 Hibernate Validator 를 사용
- Spring Boot Starter Validation 을 의존성에 추가하면 별다른 설정 없이 손쉽게 사용이 가능하다.

`Spring Boot Starter Validation`
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

#### 유효성 검사 규칙 선언

`Taco 클래스`
```java
@Data
public class Taco {

    @NotNull
    @Size(min = 5, message = "Name must be at least 5 characters long")
    private String name;

    @Size(min = 1, message = "You must choose at least 1 ingredient")
    private List<String> ingredients;
}
```

- @NotNull 애노테이션은 해당 필드에 Null 이 올 수 없음을 선언한다.
- @Size 는 해당 필드는 길이에 대한 규칙을 선언한다.
- 위 두 애노테이션 외에도 다양한 유효성검사와 관련된 애노테이션들이 존재한다.
- 자세한 사항은 아래 링크 참고
- https://beanvalidation.org/1.0/spec/

### 뷰 컨트롤러
- 모델 데이터나 사용자 입력을 처리하지 않는 간단한 컨트롤러는 다른 방식으로 정의할 수 있다.
- 뷰에 요청을 전달하기만 하는 일만 수행하는 컨트롤러를 **뷰 컨트롤러** 라고 한다.
- 뷰컨트롤러는 다음과 같이 정의할 수 있다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/").setViewName("home");
  }
}
```
- WebMvcConfigurer 는 MVC 설정을 확장할 수 있는 인터페이스를 제공한다.
- addViewControllers 를 재정의 하면 손 쉽게 뷰 컨트롤러를 등록 할 수 있다.

### 정리
- 스프링은 스프링 MVC 라는 강력한 웹 프레임워크를 제공한다.
- 애노테이션 기반으로 동작하며 @RequestMapping 과 같은 애노테이션들을 사용해 요청 처리 메소드를 선언할 수 있다.
- 유효성검사는 Hibernate Validator 와 같은 API 를 상요하 수행한다.
- 뷰에 요청을 전달만 하는 컨트롤러의 경우 뷰 컨트롤러를 사용해 처리할 수 있다.
