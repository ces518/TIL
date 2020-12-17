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
