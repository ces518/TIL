# AOP - Aspect 
- 스프링 3대 프로그래밍 모델중 하나는 AOP이다.


### AOP ? 
- Aspect-Oriendted Prgramming의 약자로 직역하면 관점지향 프로그래밍이다.
- 스프링의 DI가 의존성에 대한 주입이라면 , AOP는 로직의 주입이라고 볼수 있다.
- OOP의 한계를 보완하기 위해 나온것이다.


### 횡단관심사 ? 횡적관심사 ?
- 프로그램을 작성하다보면 다수의 모듈에 공통적으로 나타나는 부분이 존재한다.
- 이것을 Cross-Cutting concern 횡단관심사 , 횡적관심사라고한다.
- ex) 트랜잭션처리 , 로깅, 에러처리 등등..

- AOP에서 로직을 주입한다면 주입할 수 있는곳은 총 5가지이다.
    - Around (로직 전,후)
    - Before (로직 전)
    - After (로직 실행후)
    - AfterReturning (로직 종료후)
    - AfterThrowing (로직 실행중 예외가 발생한뒤 종료후)


### AOP의 용어

- Pointcut 
    - Aspect의 적용위치

- JoinPoint
    - Aspect 적용가능한 모든 지점을 의미한다.
    - 스프링 AOP에서 JoinPoint란 스프링 프레임워크가 관리하는 빈의 모든 메서드에 해당한다.

- Advice
    - pointcut에 적용할 로직, 메서드를 의미하는데 타깃 객체의 타깃 메서드에 적용될 부가기능을 의미하기도한다.

- Aspect - 관점 , 측면 Advisor의 집합체
    - AOP에서 Aspect는 여러 개의 Advice와 여러 개의 Pointcut의 결합체를 의미하는 용어다.

- Advisor
    - 한개의 Advice + 한개의 Pointcut을 의미한다.
