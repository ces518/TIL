# 더 자바, 애플리케이션을 테스트하는 방법

## Mock 객체 Stubbing
- **Stubbing** 이란 Mocking 한 객체가 어떤 행위를 할때 어떤 행동을 할지를 정하는것
- 모든 Mock 객체의 행동
    - 기본적으로 Null 을 리턴하고 Optional 타입은 Optional.empty
    - Primitive 타입은 기본 값
    - 컬렉션은 빈 컬ㄹ렉션
    - Void 메소드는 아무런 일도 발생하지 않는다.

`Stubbing`

```java
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Test
    void createStudy(
        @Mock MemberService memberService,
        @Mock StudyRepository studyRepository
    ) throws Exception {
        // given
        StudyService studyService = new StudyService(memberService, studyRepository);

        Member member = new Member();
        member.setId(1L);

        // Mocking 된 객체의 void 메소드는 기본적으로 아무런 행위도 하지 않는다.
        memberService.validate(1L);


        // Stubbing : Mocking 한 객체가 어떤 행위를 할때 어떤 행동을 할지를 정하는것
        when(memberService.findById(any())).thenReturn(Optional.of(member));

        // void method Stubbing 은 방법이 다르다.
        doThrow(RuntimeException.class).when(memberService).validate(any());


        Study study = new Study(10, "java");

        // when
        studyService.createNewStudy(1L, study);


        // then
        assertNotNull(studyService);
    }
}
```