# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 웹계층 개발_상품등록

#### 상품 등록
```java
@Getter @Setter
public class BookForm {

    private Long id;

    // == 상품 공통 속성 == //
    private String name;
    private int price;
    private int stockQuantity;
    // == == //

    private String author;
    private String isbn;
}

@Controller
@RequiredArgsConstructor
public class ItemController {

    private final ItemService itemService;

    @GetMapping("/items/new")
    public String createForm (Model model) {
        model.addAttribute("form", new BookForm());
        return "items/createItemForm";
    }

    @PostMapping("/items/new")
    public String create (BookForm form) {

        // setter를 모두 닫고, 생성자 메소드를 사용하는것이 좋은 설계임.
        Book book = new Book();
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        itemService.saveItem(book);

        return "redirect:/items";
    }
}
```

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <form th:action="@{/items/new}" th:object="${form}" method="post">
        <div class="form-group">
            <label th:for="name">상품명</label>
            <input type="text" th:field="*{name}" class="form-control"
                   placeholder="이름을 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="price">가격</label>
            <input type="number" th:field="*{price}" class="form-control"
                   placeholder="가격을 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="stockQuantity">수량</label>
            <input type="number" th:field="*{stockQuantity}" class="formcontrol"
                   placeholder="수량을 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="author">저자</label>
            <input type="text" th:field="*{author}" class="form-control"
                   placeholder="저자를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="isbn">ISBN</label>
            <input type="text" th:field="*{isbn}" class="form-control"
                   placeholder="ISBN을 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <br/>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

- 이전 회원목록과 동일하게 form 객체를 참조하여 상품 등록이 가능한 간단한 폼 화면이다.
- 상품 등록 메소드에서 Book 엔티티를 새롭게 생성하여 setter를 이용하여 데이터를 바인딩하는데 이는 좋지 못한 설계이다.
- new 로 객체생성하는것을 막고, setter를 제한한뒤 생성자 메소드를 만들어 의도한대로 엔티티 객체를 생성하도록 하는것이 좋은 설계이다.
    - 추후 유지보수에 좋음.

> 제한되도록 설계하는것이 무조건 좋은 설계라고는 단정하기 어렵지만 좋은 설계일 확률이 높다.
