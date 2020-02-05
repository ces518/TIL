# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 웹계층 개발_회원가입

#### 회원가입
`회원 컨트롤러`
```java
@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @GetMapping("/members/new")
    public String createForm (Model model) {
        model.addAttribute("memberForm", new MemberForm());
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create (@Valid MemberForm memberForm,
                          BindingResult result) {
        if (result.hasErrors()) {
            return "members/createMemberForm";
        }

        Address address = new Address(memberForm.getCity(), memberForm.getStreet(), memberForm.getZipcode());
        Member member = new Member();
        member.setName(memberForm.getName());
        member.setAddress(address);

        memberService.join(member);
        return "redirect:/";
    }
}
```

- 회원가입 폼 진입시 memberForm을 model에 담아 타임리프에서 참조할 수 있도록 한다.
- 회원 등록시 서버사이드에서 발리데이션을 진행하고, 발리데이션 에러 발생시 BindingResult를 활용해 타임리프에서 메시지를 출력해주도록 구현

#### 엔티티와 폼데이터를 분리하는 이유
- 엔티티로 바로 받아도 될텐데 폼데이터 (DTO)를 사용하는 이유는 다음과 같다.
- 1.1차적으로 엔티티와 폼데이터가 다르다.
- 2.벨리데이션 애노테이션때문에 엔티티가 지저분해진다.
> 폼데이터를 컨트롤러에서 필요한 데이터만 정제해서 엔티티로 넘기는것을 추천한다.

`회원 등록 폼`
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<style>
    .fieldError {
        border-color: #bd2130;
    }
</style>
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <!-- th:object = form:form 의 modelAttribute와 비슷 -->
    <form role="form" action="/members/new" th:object="${memberForm}" method="post">
        <div class="form-group">
            <label th:for="name">이름</label>

            <!--
                th:field: th:object 에 바인된 객체의 필드를 바인딩함
                - 기본적으로 id와 name 속성의 값을 동일하게 지정해준다.
                - ex) input type=text name=name id=name

                *{name}: memberForm 객체의 name 필드를 바인딩 한다.
                타임리프는 기본적으로 getter/setter를 활용한 프로퍼티 접근방식을 사용한다.
            -->
            <input type="text" th:field="*{name}" class="form-control"
                   placeholder="이름을 입력하세요"
                   th:class="${#fields.hasErrors('name')}? 'form-control fieldError' : 'form-control'">
            <p th:if="${#fields.hasErrors('name')}"
               th:errors="*{name}">Incorrect date</p>
        </div>
        <div class="form-group">
            <label th:for="city">도시</label>
            <input type="text" th:field="*{city}" class="form-control"
                   placeholder="도시를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="street">거리</label>
            <input type="text" th:field="*{street}" class="form-control"
                   placeholder="거리를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="zipcode">우편번호</label>
            <input type="text" th:field="*{zipcode}" class="form-control"
                   placeholder="우편번호를 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <br/>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

- th:object
    - 컨트롤러에서 model에 담은 객체를 참조하도록 지정하는 속성이다.
- th:field
    - th:object에서 참조한 객체의 필드를 바인딩한다.
    - 해당 태그의 id, name 값을 동일하게 선언해준다.

> 타임리프와 스프링의 integration이 매우 잘되어 있기때문에 폼데이터관련 처리는 매우 쉽게 가능해진다.

#### 정리
- 폼데이터 (DTO)와 엔티티는 분리하는 것이 좋다.
    - 매우 간단한 형태의 폼데이터라면 엔티티로 바로 받아도 된다.
    - 하지만 실무에서 그런 케이스는 매우 드물다.
