# 더 자바, 애플리케이션을 테스트하는 방법

## ArchUnit
- https://www.archunit.org/
- 애플리케이션의 패키지 구조 / 클래스 관계/ 참조 관계 등의 아키텍쳐를 테스트 할 수 있다.
- circular dependency 를 최대한 피하는것이 좋다.
    - 닭과 달걀 관계가 생긴다.
    - 코드가 일관된 흐름이 없고 여기저기 왔다갔다 하게 된다.
    - 클래스 / 패키지 간의 circular dependency 를 피하는것이 좋다.
- 테스트 유즈 케이스
    - A 패키지가 B, C, D 패키지에서만 사용되고 있는지 확인
    - Service 라는 이름을 가지는 클래스는 Controller / Service 에서만 사용되고 있는지 확인
    - A 애노테이션을 선언한 메소드만 특정 패키지 혹은 특정 애노테이션을 가진 클래스를 호출하고 있는지 확인
    - 특정한 스타일의 아키텍처를 따르고 있는지 확인
        - layered architecture / onion architecture ... 등등 아키텍쳐를 따르고 있는지 확인한다.
    