# Singleton Pattern
- 싱글턴 패턴이란 인스턴스를 하나만 만들어 사용하기위한 패턴이다.
- 커넥션풀, 스레드풀 등 과 같은경우  인스턴스를 여러개 만들면 불필요한 자원을 사용하게 되고, 예상치못한 결과를 낳을 수 있다.
- 오직 하나의 인스턴스만 만들어 그것을 재사용하는패턴이다.

### 제약조건
- new 를 실행할 수 없도록 생성자에 private 접근제어자를 지정한다.
- 단일 객체를 반환 할 수 있는 정적 메서드가 필요하다.
- 단일 객체를 참조할 정적 참조변수가 필요하다.

```java
public class Singleton {
    private static Singleton obj;

    private Singleton(){}

    public static Singleton getInstance() {
        return obj == null ? obj = new Singleton : obj;
    }
}
```

- "클래스의 인스턴스를 하나만 만들어 사용하는패턴"
