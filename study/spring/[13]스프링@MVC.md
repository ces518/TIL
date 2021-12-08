# 지옥 스터디 - 13 스프링 @MVC

## Spring MVC 구성요소

![img.png](images/spring_mvc.png)

## @RequestMappingHandlerMapping

- @MVC 의 가장 큰 특징 -> 기존에는 매핑의 대상이 오브젝트 였다면, **메소드** 단위로 변경되었다.
    - Ruby On Rails 와 같은 프레임워크의 영향을 많이 받음..
- @MVC 의 핸들러 매핑 처리를 위해 **DefaultAnnotationHandlerMapping (deprecated)** 를 사용한다.
    - 3.1 부터 RequestMappingHandlerMapping 으로 대체됨
- @RequestMapping 애노테이션을 매핑 정보로 활용한다.
    - 타입/메소드 레벨에 적용가능 하다.
- 최종적으로 **두 가지 위치에 적용된 정보를 결합해 최종 매핑 정보를 생성** 한다.

### @RequestMapping

- @RequestMapping 애노테이션은 매핑 정보 제공을 위한 다양한 엘리먼트들이 존재한다.
    - Spring 2.5 버전부터 추가됨
- 모든 엘리먼트들은 **생략 가능**

```java

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {

    String name() default "";

    @AliasFor("path")
    String[] value() default {};

    /**
     * The path mapping URIs (e.g. {@code "/profile"}).
     * <p>Ant-style path patterns are also supported (e.g. {@code "/profile/**"}).
     * At the method level, relative paths (e.g. {@code "edit"}) are supported
     * within the primary mapping expressed at the type level.
     * Path mapping URIs may contain placeholders (e.g. <code>"/${profile_path}"</code>).
     * <p><b>Supported at the type level as well as at the method level!</b>
     * When used at the type level, all method-level mappings inherit
     * this primary mapping, narrowing it for a specific handler method.
     * <p><strong>NOTE</strong>: A handler method that is not mapped to any path
     * explicitly is effectively mapped to an empty path.
     * @since 4.2
     */
    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};

    String[] params() default {};

    String[] headers() default {};

    String[] consumes() default {};

    String[] produces() default {};

}
```

`String[] value()`

```java

@Controller
@RequestMapping("/api/v1") // 타입 레벨에 정의, 타입 레벨에 정의 되었다면 이는 메소드레벨에 정의된 매핑 정보의 "공통 정보" 로써 사용된다.
public class SimpleController {

    // 메소드 레벨에 정의
    @RequestMapping("/users")
    public String users() {
        return "users";
    }

    /**
     * 파일확장자 패턴으로 매칭도 가능하다. 하지만 스프링 부트의 경우 기본적으로 false 로 지정되어 있다.
     * 보안상의 이슈..
     *
     * https://stackoverflow.com/questions/9688065/spring-mvc-application-filtering-html-in-url-is-this-a-security-issue
     * https://stackoverflow.com/questions/30610607/how-to-change-spring-request-mapping-to-disallow-url-pattern-with-suffix
     * https://stackoverflow.com/questions/30307678/why-does-requestmapping-spring-annotation-in-controller-capture-more-that-i-wan
     * https://stackoverflow.com/questions/22845672/requestmapping-in-spring-with-weird-patterns
     *
     * https://github.com/spring-projects/spring-framework/issues/23915
     * https://github.com/spring-projects/spring-framework/issues/24179
     *
     */
    @RequestMapping("/main.*")
    public String main() {
        return "main";
    }

    /**
     * value 속성에 명시하며, value 애트리뷰트 명은 생략가능
     */
    @RequestMapping(value = "/hello")
    public String hello() {
        return "hello";
    }

    /**
     * 배열로 하나 이상의 URL 패턴 지정도 가능하다.
     */
    @RequestMapping({"/wow", "/fantastic"})
    public String wow() {
        return "wow";
    }
}
```

- 디폴트 엘리먼트
- 스트링 배열로 URL 패턴 지정이 가능하다.
- 가장 기본이 되는 매핑 정보
- 파일 **확장자 패턴** 으로 매핑도 가능하다.
    - useSuffixPatternMatch 옵션
- 하지만 기본적으로 해당 옵션이 false 로 지정되어 있음
    - 스프링 부트의 경우 꽤나 오래전부터 false 였지만.. 스프링 MVC 의 경우에는 좀 최근에 변경됨..
    - 이는 보안상의 이슈..

![RequestMappingHandlerMapping](./images/request_mapping_handler_mapping.png)

![useSuffixPatternMatch](./images/useSuffixPatternMatch.png)

```java
@RequestMapping("/welcome")
```

- 이와 같이 매핑이 되어있다고 가정하면..
- http://www.example.com/welcome.check.blah 라고 요청을 보내도 동작하게 된다.
- 스크립트 태그가 있는 GET URL 이 있어도 요청을 보내게됨
    - http://www.example.com/welcome.<script>alert("hi")</script>

`RequestMethod[] method()`

```java
public enum RequestMethod {

    GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE

}
```

```java

@Controller
@RequestMapping("/api/v1") // 타입 레벨에 정의, 타입 레벨에 정의 되었다면 이는 메소드레벨에 정의된 매핑 정보의 "공통 정보" 로써 사용된다.
public class SimpleController {

    /**
     * 요청 메소드 지정도 가능하다 (배열로 지정가능)
     * 동일한 요청 URL 이더라도 메소드가 다르다면, 다른 매핑으로 인지한다.
     * @see org.springframework.web.bind.annotation.RequestMethod
     */
    @RequestMapping(value = "/ncucu", method = RequestMethod.GET)
    public String requestMethod() {
        return "requestMethod";
    }

    /**
     * 메타 애노테이션으로 활용한 GetMapping, PostMapping 등이 편의를 위해 추가됨 (스프링 4.3)
     */
    @GetMapping("/ncucu2")
    public String requestMethod2() {
        return "requestMethod2";
    }
}
```

- HTTP 요청 메소드를 지정
- RequestMethod 는 HTTP 메소드를 정의한 enum
- 배열로 지정이 가능하고, 동일한 URL 이더라도 요청 메소드가 다르다면 다른 매핑으로 인지한다.
- 스프링 4.3 부터 @GetMapping, @PostMapping 과 같은 편의를 위한 메타 애노테이션이 추가되었다.

```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RequestMapping(method = RequestMethod.GET)
public @interface GetMapping {

    @AliasFor(annotation = RequestMapping.class)
    String name() default "";

    @AliasFor(annotation = RequestMapping.class)
    String[] value() default {};

    @AliasFor(annotation = RequestMapping.class)
    String[] path() default {};

    @AliasFor(annotation = RequestMapping.class)
    String[] params() default {};

    @AliasFor(annotation = RequestMapping.class)
    String[] headers() default {};

    @AliasFor(annotation = RequestMapping.class)
    String[] consumes() default {};

    @AliasFor(annotation = RequestMapping.class)
    String[] produces() default {};

}
```

`String[] params()`

```java

@Controller
@RequestMapping("/api/v1") // 타입 레벨에 정의, 타입 레벨에 정의 되었다면 이는 메소드레벨에 정의된 매핑 정보의 "공통 정보" 로써 사용된다.
public class SimpleController {

    /**
     * 요청 파라미터와 값을 비교해 매핑하는 방식
     * 동일한 URL 이더라도, 요청 파라미터에 따라 별도의 작업을 해 줄 수 있다.
     */
    @GetMapping(value = "params", params = "type=admin")
    public String admin(String type) {
        return "admin";
    }

    @GetMapping(value = "params", params = "type=member")
    public String member(String type) {
        return "member";
    }

    /**
     * 특정 파라미터가 존재해선 안된다는 매핑 선언도 가능
     */
    @GetMapping(value = "params", params = "!type")
    public String notType(String type) {
        return "notType";
    }
}
```

- 요청 파라미터와 값을 비교해 매핑 정보를 제공
- 동일 URL 이더라도, 요청 파라미터에 따라 별도의 작업을 할 수 있다.

`String[] headers()`

```java

@Controller
@RequestMapping("/api/v1") // 타입 레벨에 정의, 타입 레벨에 정의 되었다면 이는 메소드레벨에 정의된 매핑 정보의 "공통 정보" 로써 사용된다.
public class SimpleController {

    /**
     * 특정 헤더와 키/값이 동일할때만 매핑해준다.
     * 요청 파라미터와 유사하다
     */
    @GetMapping(value = "header", headers = "content-type=text/*")
    public String headers() {
        return "";
    }
}

```

- HTTP 헤더 정보를 활용한 매핑 정보를 제공
- 자주 사용되지는 않는다.

### 메소드/타입 레벨 매핑

- @RequestMapping 은 메소드 단위로 핸들러를 매핑하는 방식
- 메소드/타입 레벨에 매핑정보를 지정할 수 있다.
- 메소드 단위 매핑중에 공통적인 정보가 있다면, 타입레벨로 이를 추출해 해당 클래스 전역으로 활용도 가능하다.
    - 공통적인 매핑 정보가 없다면, 타입레벨 매핑은 생략을 해도 무관하다.
    - 대부분의 경우는 UserController / ProductController 이런 식으로 구성을 해서 사용할 것이기 때문에 메소드 + 타입 레벨 매핑을 주로 사용함..

```java

@Controller
@RequestMapping("/api/v1") // 타입 레벨에 정의, 타입 레벨에 정의 되었다면 이는 메소드레벨에 정의된 매핑 정보의 "공통 정보" 로써 사용된다.
public class SimpleController {

    // 메소드 레벨에 정의
    @RequestMapping("/users")
    public String users() {
        return "users";
    }
}
```

### 타입 상속과 매핑

- @RequestMapping 이 적용된 클래스를 **상속** 해서 사용한다면 슈퍼 클래스의 정보는 어떻게 될까 ?
    - 언어차원에서 봤을때 애노테이션 정보는 기본적으로 상속되지 않는다. (@Inherited 가 적용되지 않은 경우)
- 하지만 **프레임워크** 레벨로 들어갔을때는 조금 다르다.
- 부모 클래스에 적용된 애노테이션까지 처리하는 경우가 많음.. (Spring/JPA 같은...)
- 스프링의 @RequestMapping 은 상속된다 라고 이해해도 좋다.
    - 프레임워크 레벨에서 부모 클래스에 적용된 애노테이션 정보까지 활용하기 때문
- 매핑 정보 상속과 종류의 대표적인 경우를 몇가지 살펴보자...

`상위 타입과 메소드의 @RequestMapping`

```java

@RequestMapping("/user")
public class Super {

    @RequestMapping("/list")
    public String list() {
        return "";
    }
}

/**
 * 클래스 상속의 경우 슈퍼 클래스의 매핑정보가 자식에게 그대로 적용된다.
 * 매핑정보가 명시된 슈퍼 클래스의 메소드를 오버라이드 했더라도, 자식클래스에 매핑정보를 명시하지 않는한 해당 정보도 상속된다.
 */
public class Sub extends Super {

    @Override
    public String list() {
        return super.list();
    }
}
```

- 부모 클래스에만 @RequestMapping 을 적용하고, 이를 상속한 자식 클래스에서는 아무런 적용도 하지 않았을 경우이다.
- 클래스 상속의 경우 부모 클래스의 매핑 정보가 자식 클래스에게 그대로 적용된다.
- 하위 클래스가 해당 메소드를 오버라이드 하더라도, 자식 클래스에 매핑 정보를 명시하지 않는한, 해당 정보도 **상속** 된다.
  - 인터페이스 구현의 경우도 동일하다.

`상위 타입의 @RequestMapping 과 하위 타입 메소드의 @RequestMapping`

```java
@RequestMapping("/user")
public class Super {


    public String list() {
        return "";
    }
}


public class Sub extends Super {

    @RequestMapping("/list")
    @Override
    public String list() {
        return super.list();
    }
}
```

- 부모 클래스의 타입에만 @RequestMapping 이 적용되어 있고, 자식 클래스에는 메소드에만 @RequestMapping 이 적용된 경우이다.
- 부모 타입레벨 매핑정보 + 자식 메소드레벨 매핑정보가 결합되어 최종적인 매핑 정보가 생성된다.
  - 인터페이스 구현의 경우도 동일하다.

`상위 타입 메소드의 @RequestMapping 과 하위 타입의 @RequestMapping`
- 이전 케이스와 반대되는 경우이다.
- 부모 클래스에는 메소드 레벨에만 적용되었고, 자식 클래스는 타입 레벨에만 적용된 경우..
- 이도 마찬가지로 매핑정보가 상속되어 결합된다.
  - 인터페이스 구현의 경우도 동일하다.

`하위 타입과 메소드의 @RequestMapping 재정의`

```java
@RequestMapping(value = "/user", method = RequestMethod.POST)
public class Super {

    @RequestMapping("/list")
    public String list() {
        return "";
    }
}

@RequestMapping("/ncucu")
public class Sub extends Super {

    @RequestMapping("/me")
    @Override
    public String list() {
        return super.list();
    }
}
```
- 자식 클래스가 @RequestMapping 을 적용하면 부모의 매핑정보를 대체해서 적용된다.
  - 이는 타입/메소드 레벨 모두 적용된다.
- 조금 주의할 점은, @RequestMapping 이 통으로 오버라이드 된다고 이해하면 좋다.
  - 예제에서 부모 클래스에 정의된 RequestMethod 매핑 정보는 사라짐..

`하위 타입 메소드의 URL 패턴이 없는 @RequestMapping 재정의`
- 자식 클래스에서 재정의한 @RequestMapping 은 부모 클래스에 정의한 매핑 정보를 대체한다.
- 하지만 조금 특이한 케이스는, **클래스 상속** 에 한해 오버라이드한 하위 메소드에 한해 URL 매핑 정보가 없다면 이는 무시된다.

> 다양한 매핑 정보 상속을 알아봤는데... 사실 실무에서 사용하는 경우는 거의 없을 거라고 생각함.. <br/>
> 사내 프레임워크를 만든다거나 하지 않는한 거의 안쓰이는 부분이고 사실 사용한다고 해도 가능하면 명시적으로 사용하는걸 권장하고 싶음... <br/>
> 의도치 않게 동작할 수도 있고 모든 팀원이 이걸 다 알아야 제대로 사용이 가능하다고 생각..

## 참고

- https://github.com/spring-projects/spring-framework/issues/23915
- https://github.com/spring-projects/spring-framework/issues/24179
- https://stackoverflow.com/questions/9688065/spring-mvc-application-filtering-html-in-url-is-this-a-security-issue
- https://stackoverflow.com/questions/30610607/how-to-change-spring-request-mapping-to-disallow-url-pattern-with-suffix
- https://stackoverflow.com/questions/30307678/why-does-requestmapping-spring-annotation-in-controller-capture-more-that-i-wan
- https://stackoverflow.com/questions/22845672/requestmapping-in-spring-with-weird-patterns
- https://hamait.tistory.com/314