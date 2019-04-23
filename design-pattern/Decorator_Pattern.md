# Decoprator Pattern
- 데코레이터는 장식자를 의미한다.
- 데코레이터 패턴은 프록시패턴과 구현방법이 같다.
- 데코레이터 패턴은 클라이언트가 받는 반환값에 장식을 덧입히는 패턴이다.

```java
public class ServiceImpl implements Serivce {
    public String run() {
        return "service";
    }
}

public class Decorator implements Service {
    Service service;

    public String run() {
        //데코레이터 로직
        return "데코레이터 패턴 : " + service.run();
    }
}
```

- 반환값에 장식을 더한다는것을 제외하면 프록시패턴과 동일하다.
- "메서드 호출의 반환값에 변화를 주기위해 중간에 장식자를 두는 패턴"
