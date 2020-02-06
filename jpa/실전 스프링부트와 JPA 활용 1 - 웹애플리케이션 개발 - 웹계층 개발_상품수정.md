# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 웹계층 개발_상품수정

#### 상품 수정
```java
@PostMapping("/items/{itemId}/edit")
public String updateItem (@ModelAttribute("form") BookForm form) {

    Book book = new Book();
    book.setId(form.getId());
    book.setName(form.getName());
    book.setPrice(form.getPrice());
    book.setStockQuantity(form.getStockQuantity());
    book.setAuthor(form.getAuthor());
    book.setIsbn(form.getIsbn());

    itemService.saveItem(book);
    return "redirect:/items";
}
```

> 데이터수정시 JPA에서는 변경감지를 병합(merge) 보다는 변경감지를 사용하는것이 베스트 프렉티스이다.
