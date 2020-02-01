# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 웹계층 개발

#### 웹 계층 개발
- Thymeleaf 사용

`순서`
- 홈 화면과 레이아웃
- 회원 등록
- 회원 목록조회
- 상품 등록
- 상품 목록
- 상품 수정
- 변경 감지와 병합(merge)
- 상품 주문
- 주문 목록 검색,취소


#### 홈 화면과 레이아웃

`홈 컨트롤러`
```java
@Slf4j
@Controller
@RequiredArgsConstructor
public class HomeController {

    // 롬복으로 대체
//    Logger loger = LoggerFactory.getLogger(HomeController.class);

    @RequestMapping("/")
    public String home () {
        log.info("home controller .. ");
        return "home";
    }
}
```

- '/' 루트로 요청이 들어오면 resources/template/home.html로 viewResolver가 포워딩을 해주는 역할을 한다.

`home.html`
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header">
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader" />
    <div class="jumbotron"> <h1>HELLO SHOP</h1>
        <p class="lead">회원 기능</p> <p>
            <a class="btn btn-lg btn-secondary" href="/members/new">회원 가입</a>
            <a class="btn btn-lg btn-secondary" href="/members">회원 목록</a> </p>
        <p class="lead">상품 기능</p> <p>
            <a class="btn btn-lg btn-dark" href="/items/new">상품 등록</a>
            <a class="btn btn-lg btn-dark" href="/items">상품 목록</a> </p>
        <p class="lead">주문 기능</p> <p>
            <a class="btn btn-lg btn-info" href="/order">상품 주문</a>
            <a class="btn btn-lg btn-info" href="/orders">주문 내역</a> </p>
    </div>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
```

- 템플릿 엔진을 thyleaf로 사용한다.
- 다른부분들은 문제가 되지 않을것 같은데 **th:replace 라는 구문이 등장한다.** 
- Tiles 와 같은 뷰 프레임워크 처럼 여러 파일로 쪼개두고 replace 할 수 있는 기능을 제공하는 구문이다.
- 즉 fragments 폴더에 header를 include 시킨다. (재사용성)

`header.html`
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="header">
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-
  to-fit=no">
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="/css/bootstrap.min.css" integrity="sha384-
  ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T"
          crossorigin="anonymous">
    <!-- Custom styles for this template -->
    <link href="/css/jumbotron-narrow.css" rel="stylesheet">
    <title>Hello, world!</title>
</head>
```

`bodyHeader.html`
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<div class="header" th:fragment="bodyHeader">
    <ul class="nav nav-pills pull-right">
        <li><a href="/">Home</a></li>
    </ul>
    <a href="/"><h3 class="text-muted">HELLO SHOP</h3></a>
</div>
```

`footer.html`
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<div class="footer" th:fragment="footer">
    <p>&copy; Hello Shop V2</p>
</div>
```

include-style-layouts
- include 레이아웃

Hierachy-style-layouts
- 계층형 레이아웃

Bootstrap
- https://getbootstrap.com/docs/4.4/getting-started/download/
