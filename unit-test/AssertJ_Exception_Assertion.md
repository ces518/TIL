# AssertJ Exception Assertion
- AssertJ 의 Excetpion Assertion 기능 제공
- catchThrowable(new ThrowableAssert.ThrowingCallable(){}) 내부에
- 에러가 발생할 테스트 로직을 작성한다
- assertThat(thrown)
        .isInstanceOf(Exception class); 를 assertion 한다
- JDK1.8이상이라면 람다로 구현이가능하다.

```java
//when
Throwable thrown = catchThrowable(new ThrowableAssert.ThrowingCallable() {
    @Override
    public void call() throws Throwable {
        MecPolicyVo mecPolicyVo = new MecPolicyVo();
        mecPolicyVo.setSeq(TEST_SEQ);
        policyService.findPolicy(mecPolicyVo);
    }
});

//then
assertThat(thrown)
        .isInstanceOf(NotFoundContentsException.class);
```
