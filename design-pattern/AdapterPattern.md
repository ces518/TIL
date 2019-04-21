# Adapter Pattern (어댑터 패턴)
- 어댑터를 변환하면 변환기라고 할 수 있다.
- 변환기의 역할은 서로 다른 두 인터페이스 사이에 통신이 가능하게 하는것이다.
- Java 의 JDBC가 어댑터 페턴의 좋은 예이다.

아래는 어댑터 패턴이 적용되지 않은 코드이다.
```java
public class AService {
    void runA() {
        System.out.println("A");
    }
}

public class BService {
    void runB() {
        System.out.println("B");
    }
}


public class Main {
    public static void main(String[] args) {
        AService a = new AService();
        BService b = new BService();

        a.runA();
        b.runB();
    }
}

```

#### 문제점 ?
- A서비스와 B서비스의 실행 메서드가 다르기때문에 A 서비스를 이용하다가 B로 변경한다고 한다면 , 해당 서비스를 이용하는 모든 곳의 코드를 수정해주어야한다.


#### 어댑터 패턴의 적용

아래는 어댑터 패턴을 적용하여 메서드명을 통일 한 것이다.

```java
public class AdapterAService {
    AService a = new AService();

    void run() {
        a.runA();
    }
}

public class AdapterBService {
    BService b = new BService();

    void run() {
        a.runB();
    }
}

public class MainWithAdapter {
    public static void main(String[] args) {
        AdapterAService a = new AdapterAService();
        AdapterBService b = new AdapterBService();

        a.run();
        b.run();
    }
}

```

- 클라이언트가 변환기 (Adapter) 를 통해 run() 이라는 동일한 메서드 명으로
두 객체의 메서드를 호출한다.

- 어댑터 패턴은 합성, 즉 객체를 속성으로 만들어 참조하는 디자인 패턴이다.

"호출 당하는 쪽의 메서드를 호출하는 쪽의 코드에 대응하도록 중간 변환기를 두는 패턴"
