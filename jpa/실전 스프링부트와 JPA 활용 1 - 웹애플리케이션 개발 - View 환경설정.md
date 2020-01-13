# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - View 환경설정

#### View 환경설정
- Thymeleaf 템플릿 엔진 
- thymeleaf.org
- 스프링 공식 튜토리얼: https://spring.io/guides/gs/serving-web-content/

> 가급적이면 JSP 보다는 템플릿 엔진을 사용하는것을 권장

Thymeleaf, Freemarker, Mustache 등..

`장점`
타임리프의 장점은 markup 을 깨지않는 템플릿 엔진 (웹브라우저에서 단독으로 실행이 가능하다.)

`단점`
Thymeleaf 2.x 때는 closing 방식을 채용해야 하기 때문에 불편한점이 존재했지만 3.x부터 이런 방식이 개선되고, 성능문제도 많이 해결되었다.

> 가능하다면 서버사이드 랜더링 보다는 React, VueJS 등을 사용할것을 추천한다.

#### Thymeleaf 매핑
- 기본경로
    - resources/templates/**.html
```java
@Controller
public class HelloController {

    @GetMapping("hello")
    public String hello (Model model) {
        model.addAttribute("data", "hello!!");
        return "hello";
    }
}
```
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>HELLO</title>
</head>
<body>
<p th:text="'안녕하세요,' + ${data}" >안녕하세요, 손님</p>

</body>
</html>
```

> 정적인 페이지는 resources/static, 템플릿엔진은 resources/templates가 기본 경로이다. (스프링부트 사용시 기본설정)

resources 폴더 하위에 존재하는 파일은 기본적으로 수정을 할때마다 재시작을 해주어야 한다.

#### Spring Boot devtools
- 템플릿 파일 수정시마다 매번 서버를 재시작하는 것도 매우 번거로운 일이다.
- 스프링 부트에서는 개발시 편의성을 제공하기 위해 devtools 라는 라이브러리를 제공한다.
- spring-boot-starter-devtools 를 dependency로 추가한뒤, 서버를 재시작했을때 로그화면에 다음과 같이 [restartedMain]이 존재한다면 devtools세팅이 제대로 동작한 것이다.
```java
2020-01-13 20:07:23.737  INFO 1173 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2020-01-13 20:07:23.792  WARN 1173 --- [  restartedMain] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2020-01-13 20:07:23.911  INFO 1173 --- [  restartedMain] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-01-13 20:07:23.975  INFO 1173 --- [  restartedMain] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page: class path resource [static/index.html]
2020-01-13 20:07:24.135  INFO 1173 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-01-13 20:07:24.139  INFO 1173 --- [  restartedMain] jpabook.jpashop.JpashopApplication       : Started JpashopApplication in 17.843 seconds (JVM running for 23.486)
```

> devtools 세팅이 되어있다면, templates/hello.html 파일을 수정한뒤, 해당 파일만 다시 빌드해주면 서버 재시작없이 수정된 사항이 바로 반영되는것을 확인할 수 있다.
