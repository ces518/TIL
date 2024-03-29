# 더 자바, 애플리케이션을 테스트하는 방법

## Mockito
- Mock 객체란 ?
    - 실제 객체와 유사하지만 개발자가 코드레벨에서 해당 객체의 행동을 컨트롤 할 수 있다.
- Mockito
    - Mock 객체를 생성 , 관리 및 검증하기 쉬운 프레임워크
- 단위 테스트에 대한 고찰
    - https://martinfowler.com/bliki/UnitTest.html
- 일부 개발자는 단위 테스트에 대해서 단위를 strict 하게 하는 사람이 있다.
    - 하지만 반드시 그래야만 UnitTest 라고 하진 않는다.
    - 이 단위를 클래스, 오브젝트 단위로만 생각하는게 아니라, '행동' 의 단위
    - 해당 행동과 연관되는 객체들은 한번에 테스트 되어도 된다고 생각..

- '단위' 를 행동을 단위로 잡는다면 관련된 객체들은 한꺼번에 테스트 되어도 된다.
- 하지만 외부 API 등을 호출해야 하는 경우 mocking 이 필요할 수 있다.

> EasyMock, JMock 등도 많이 사용된다.