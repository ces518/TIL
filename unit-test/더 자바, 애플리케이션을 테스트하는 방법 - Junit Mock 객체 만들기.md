# 더 자바, 애플리케이션을 테스트하는 방법

## Mock 객체 만들기
- Mockito 를 사용해서 Mock 객체를 만드는 방법은 크게 **코드 레벨에서 생성하는 방법** 과, **애노테이션을 사용하는 방법** 두가지로 나뉜다.

`코드 레벨에서 생성하는 방법`

```java
class StudyServiceTest {

    @Test
    void createStudyService() throws Exception {
        // given
        MemberService memberService = mock(MemberService.class);
        StudyRepository studyRepository = mock(StudyRepository.class);

        // when
        StudyService studyService = new StudyService(memberService, studyRepository);

        // then
        assertNotNull(studyService);
    }
}
```
- Mockito.mock 메소드를 활용하여 Mock 객체를 생성할 수 있다.

`애노테이션을 사용해서 생성하는 방법`

```java
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    // MockitoExtension 이 존재해야 한다.
    @Mock MemberService memberService;

    @Mock StudyRepository studyRepository;


    @Test
    void createStudyService(
        // 특정 메소드에서만 쓰인다면, 메소드 파라미터로 선언해서 스코프를 제한할 수 있다.
        @Mock MemberService memberService,
        @Mock StudyRepository studyRepository
    ) throws Exception {
        // when
        StudyService studyService = new StudyService(memberService, studyRepository);

        // then
        assertNotNull(studyService);
    }
}
```
- 애노테이션을 사용하는 방법은 2가지로 나뉘는데, 필드로 선언하는 방법과, 메소드 파라미터로 선언하는 방법 두 가지로 나뉜다.
- 만약 특정 메소드에서만 쓰이는 객체를 Mocking 한다면 메소드 파라미터로 선언하여 스코프를 제한할 수 있다.

> @Mock 애노테이션을 사용해서 Mock 객체를 생성하는 방법은 반드시 MockitoExtension 이 존재해야 한다 