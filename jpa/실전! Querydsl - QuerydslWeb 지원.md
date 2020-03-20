# 실전! Querydsl - QuerydslWeb 지원

#### QuerydslWeb 지원
- https://docs.spring.io/spring-data/jpa/docs/2.2.5.RELEASE/reference/html/#core.web

```java
@Controller
class UserController {

  @Autowired UserRepository repository;

  @RequestMapping(value = "/", method = RequestMethod.GET)
  String index(Model model, @QuerydslPredicate(root = User.class) Predicate predicate,    
          Pageable pageable, @RequestParam MultiValueMap<String, String> parameters) {

    model.addAttribute("users", repository.findAll(predicate, pageable));

    return "index";
  }
}
```

- Predicate를 생성해서 파라메터의 조건에 맞게 바인딩을 해준다.
  - ?firstname=Dave&lastname=Matthews
  
`한계점`
- 이 기능은 사실상 equal 조건만 가능하다.
- QuerydslBinderCustomizer 를 사용해서 커스터마이징이 가능하지만 한계가 너무 명확하다.
- 컨트롤러가 Querydsl에 의존한다.

> 실무에서 사용하지 못한다.
