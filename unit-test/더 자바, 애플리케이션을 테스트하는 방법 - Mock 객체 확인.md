# 더 자바, 애플리케이션을 테스트하는 방법

## Mock 객체 확인
- Mock 객체를 검증하는 기능을 제공
- verify()
  - 특정 메소드가 특정한 매개변수로 몇번 호출되었는가, 전혀 호출되지 않았는가
  - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#exact_verification
- inOrder() 
  - 순서대로 호출 되었는가
  - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#in_order_verification
- timeout()
  - 특정 시간 내에 호출 되었는가
  - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#verification_timeout
- verifyNoMoreInteractions()
  - 특정 시점 이후에는 호출되지 않았는가
  - https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#finding_redundant_invocations

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

        when(memberService.findById(any())).thenReturn(Optional.of(member));

        Study study = new Study(10, "java");

        // when
        Study newStudy = studyService.createNewStudy(1L, study);


        // then

        assertEquals(study.getOwner(), member);

        // 특정 메소드 호출
        verify(memberService).notify(newStudy);

        // 아무것도 하지 않는지 검증
        verifyNoInteractions(member);

        // 순서대로 호출이 되었는지 검증
        InOrder inOrder = inOrder(memberService);
        inOrder.verify(memberService).notify(newStudy);
    }
}
```