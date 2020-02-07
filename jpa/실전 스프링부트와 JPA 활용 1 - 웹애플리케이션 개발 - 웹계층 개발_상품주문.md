# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 웹계층 개발_상품주문

#### 상품 주문

```java
@PostMapping("/order")
public String order (@RequestParam Long memberId,
                        @RequestParam Long itemId,
                        @RequestParam int count) {

    // 엔티티를 조회해서 넘기는 것보다 식별자를 넘기는 이유 ?
    // 컨트롤러에서 찾으면 컨트롤러 로직이 지저분해지는 것도 있지만
    // 아이디값만 넘겨서 트랜잭션 내부에서 엔티티를 조회하는것이 테스트 작성시에도 좋음
    // 서비스계층에서 더 많은 것을 할수 있기 때문
    // 트랜잭션에 있는 서비스 계층의 비즈니스로직에서 사용될때 누릴수 있는 혜택이 많음
    orderService.order(memberId, itemId, count);

    return "redirect:/orders";
}
```

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <form role="form" action="/order" method="post">
        <div class="form-group">
            <label for="member">주문회원</label>
            <select name="memberId" id="member" class="form-control">
                <option value="">회원선택</option>
                <option th:each="member : ${members}"
                        th:value="${member.id}"
                        th:text="${member.name}" />
            </select>
        </div>
        <div class="form-group">
            <label for="item">상품명</label>
            <select name="itemId" id="item" class="form-control">
                <option value="">상품선택</option>
                <option th:each="item : ${items}"
                        th:value="${item.id}"
                        th:text="${item.name}" />
            </select>
        </div>
        <div class="form-group">
            <label for="count">주문수량</label>
            <input type="number" name="count" class="form-control" id="count"
                   placeholder="주문 수량을 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <br/>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```


##### 컨트롤러에서 엔티티를 넘기지않고 식별자를 넘기는 이유 ?
- 컨트롤러에서 조회해서 넘길경우 컨트롤러의 로직이 지저분해지는 것도 있지만
- 식별자만 넘겨 트랜잭션 내부에서 엔티티를 조회하는것이 테스트 작성시에도 좋음
- 또한 서비스 계층에서 많은것을 할 수 있기 때문에 서비스 계층에서 엔티티를 조회하는것을 추천한다.
    - 트랜잭션에 있는 서비스 계층의 비즈니스로직에서 사용될때 누릴 수 있는 혜택이 많다.
