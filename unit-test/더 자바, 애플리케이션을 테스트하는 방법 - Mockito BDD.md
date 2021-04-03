# 더 자바, 애플리케이션을 테스트하는 방법

## Mockito BDD
- BDD 는 애플리케이션이 어떻게 "행동" 해야 하는지에 대한 공통된 이해를 구성하는 방법 TDD 로 부터 파생되었다.
- 행동에 대한 스펙
  - Title
  - Narrative
    - As a / i want / so that
      - ex) 스터디에 참석하고 싶다면 / 참석할 수 있다.
    - Acceptance criteria
      - Given / When / Then
  
> Given When Then 만 사용한다고 해서 BDD 는 아니다.

```java
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Test
    void createStudy(
        // 특정 메소드에서만 쓰인다면, 메소드 파라미터로 선언해서 스코프를 제한할 수 있다.
        @Mock MemberService memberService,
        @Mock StudyRepository studyRepository
    ) throws Exception {
        // given
        StudyService studyService = new StudyService(memberService, studyRepository);

        Member member = new Member();
        member.setId(1L);

        // Stubbing : Mocking 한 객체가 어떤 행위를 할때 어떤 행동을 할지를 정하는것
        // BDD Style 은 when -> given , then -> willReturn
        // BDDMockito 클래스로 제공됨
        given(memberService.findById(any())).willReturn(Optional.of(member));

        Study study = new Study(10, "java");

        // when
        Study newStudy = studyService.createNewStudy(1L, study);


        // then

        assertEquals(study.getOwner(), member);

        // 특정 메소드 호출
        verify(memberService).notify(newStudy);

        // BDD Style verify -> then
        then(memberService).should().notify(newStudy);

        // 아무것도 하지 않는지 검증
        verifyNoInteractions(memberService);

        then(memberService).shouldHaveNoMoreInteractions();

        // 순서대로 호출이 되었는지 검증
        InOrder inOrder = inOrder(memberService);
        inOrder.verify(memberService).notify(newStudy);
    }
}
```
- when() -> given()
- then() -> willReturn()
- verify() -> then()

> BDD, TDD 이런 방법론에 얽매이기 보다는 **테스트코드를 작성 하는 것** 이 우선되어야 한다.\
> 테스트 코드 작성이 자리잡혀 있지도 않은데 방법론 부터 추구한다면 순서가 잘못된 것