# Proxy Pattern (프록시 패턴)
- 프록시는 대리자 를 의미한다.
- 다른 누군가를 대신해 그 역할을 수행하는 존재를 의미한다.

아래는 프록시패턴을 적용하지 않은 코드이다.
```java
public class Service {
    public String runService() {
        return "서비스 호출";
    }
}

public class Main {
    public static void main(String[] args) {
        Service service = new Serice();
        
        System.out.println(service.runSerice());
        // 서비스호출
    }
}

```

- 프록시 패턴의 경우 실제 서비스 객체가 가진 메서드와 같은 이름의 메서드를 사용하는데, 이를 위해서 인터페이스를 사용한다.

- 인터페이스를 사용하면 , 서비스 객체가 들어갈 자리에 프록시 객체를 대신 투입해
클라이언트 쪽에서는 어떤 객체의 메서드를 호출하는지 전혀 모르게 처리할 수 도 있다.

```java
public interface Serice{
    String runService();
}

public class ServiceImpl implements Service {
    public String runService() {
        return "서비스 호출";
    }
}

public class Proxy implements Service {
    Serivce service;

    public String runService() {
        System.out.println("호출에 대한 흐름 제어가 목적 이다, 반환결과를 그대로 전달한다. ");

        service = new ServiceImpl();

        return service.runService();
    }
}

public class MainWithProxy {
    public static void main(String[] args) {
        Serivce proxy = new Proxy();

        System.out.println(proxy.runService);
    }
}
```

* 프록시 패턴의 중요포인트
    - 프록시 객체는 실제 서비스와 같은 이름의 메서드를 구현한다. (인터페이스 사용)
    - 프록세는 실제 서비스에 대한 참조변수를 가진다 (합성)
    - 프록시는 실제 서비스와 같은 이름을 가진 메서드를 호출하고 , 그 값을 클라이언트에게 전달한다.
    - 프록시는 실제 서비스의 메서드 호출 전후에 별도의 로직을 수행 할 수 있다.

* 프록시 패턴은 실제 서비스의 메서드를 수정하지않고 , 제어의 흐름을 변경하거나
다른 로직을 수행하기위해 사용한다.

- 개방폐쇄의 원칙과 의존성 역전의 원칙이 적용된 디자인패턴이다.
- AOP
