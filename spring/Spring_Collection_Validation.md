# Spring Collection Validation
- JavaBeans Class 가 아닌 Collection class를 validation 하려고하면
- validation이 적용되지않는다.

#### 이유는 ?
- @Valid는 JavaBeans에 적용되는데, List는 JavaBeans가 아니기 때문이다(참고: http://stackoverflow.com/a/35643761)

#### 문제 
- Collection 형태의 HttpRequest body 데이터를 @Valid로 검증하기 위해,
- Custom Validator를 쓰면서도, DTO에 지정된 @NotNull 등을 그대로 사용하고,
- Validation이 실패하면 실패 이유를 로그에 남기고 싶다.

#### 해결방안 
- 다음과 같이 customValidator를 생성한다.
```java
public class CategoriesValidator implements Validator {

    private SpringValidatorAdapter validatorAdapter;

    public CategoriesValidator() {
        this.validatorAdapter = new SpringValidatorAdapter(
                Validation.buildDefaultValidatorFactory().getValidator()
        );
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return Collection.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object o, Errors errors) {
        Collection collection = (Collection) o;

        for (Object object: collection) {
            validatorAdapter.validate(object,errors);
        }
    }
}
````

- Controller 에서 다음과같이 사용
```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/categories")
public class MecCategoryRestController {

    private final MecCategoryService categoryService;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private CategoriesValidator validator;

    @PutMapping
    public ResponseEntity updateCategory(@Valid @RequestBody List<MecCategoryDto.Update> dtoList, BindingResult result) {
        validator.validate(dtoList,result);
        if(result.hasErrors()) {
            ErrorReponse errorReponse = new ErrorReponse(ErrorCode.BAD_REQUEST, result.getFieldErrors());
            return  ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errorReponse);
        }

        for (MecCategoryDto.Update updateDto: dtoList) {
            MecCategoryVo mecCategoryVo = objectMapper.convertValue(updateDto, MecCategoryVo.class);
            categoryService.updateCategory(mecCategoryVo);
        }
        return ResponseEntity.ok().build();
    }

}
```
