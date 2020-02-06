# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 웹계층 개발_회원목록

#### 회원목록 조회

`회원 목록`
```java
@GetMapping("/members")
public String list (Model model) {
    List<Member> members = memberService.findMembers();
    model.addAttribute("members", members);
    return "members/memberList";
}
```


`memberList.html`
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader" />
    <div>
        <table class="table table-striped">
            <thead>
            <tr>
                <th>#</th>
                <th>이름</th>
                <th>도시</th>
                <th>주소</th>
                <th>우편번호</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="member : ${members}">
                <td th:text="${member.id}"></td>
                <td th:text="${member.name}"></td>
                <td th:text="${member.address?.city}"></td><!-- ?문법은 널체크문법. -->
                <td th:text="${member.address?.street}"></td>
                <td th:text="${member.address?.zipcode}"></td>
            </tr>
            </tbody>
        </table>
    </div>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```
- Null 체크와 같은 부분에서 지저분해질수 있기 때문에 타임리프에서는 ? 문법을 제공한다.
    - address?.city
    - address가 널이면 city 프로퍼티에 접근하지 않아 nullpointer가 발생하지 않음

> 타임리프의 장점은 html 엘리멘트에 바로 적용해서 쓸 수 있다는 점이다.

#### 참고
- 엔티티를 최대한 순수하게 유지해야한다.
    - 핵심 비지니스로직만 존재하도록 설계하는 것이 중요하다.
    - 유지보수성을 위함
- 화면 종속적인 기능이 엔티티에 게속 추가되기때문에 엔티티가 지저분해진다
- 화면에 출력할때도 DTO를 가지고 화면에 꼭 필요한 데이터만 model에 담는것이 좋다
- 서버사이드 랜더링시에는 크게 문제가 되지 않지만, API를 만들떄는 절때 외부에 엔티티를 반환해서는 안된다.
    - 멤버엔티티에 필드가 추가되면 api 스펙이 변경되어 버린다.
