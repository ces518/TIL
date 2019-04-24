# Template Method Pattern
- 강아지 클래스와 , 고양이 클래스가 있다고 가정하고 , 해당 애완동물들과 즐거운 시간을 보내는것을
프로그램으로 표현한다면 다음의 코드가 될것이다.

```java
public class Dog {
    public void play() {
        System.out.println("이리온");
        System.out.println("멍멍");
        System.out.println("살랑살랑 ~ ");
    }
}

public class Cat {
    public void play() {
        System.out.println("이리온");
        System.out.println("야옹야옹");
        System.out.println("살랑살랑 ~ ");
    }
}
```

- 동물의 울음소리를 내는부분만 제외하고는 모두 일치하는것을 볼 수 있다.
- 객체지향 4대 특성중 상속을 활용하여 중복은 상위클래스로 , 닫른부분만 하위클래스로 구현하여 코드를 개선해보자.

```java
public abstract class Animal {
    public void play() {
        System.out.println("이리온");
        cry();
        doSomething();
        System.out.println("잘했어 ");  
    }

    abstract void cry();

    void doSomething() {
        System.out.println("살랑살랑 ~ ");  
    }
}

public class Dog extends Animal {
    @Override
    public void cry() {
        System.out.println("멍멍");  
    }
}

public class Cat extends Animal {
    @Override
    public void cry() {
        System.out.println("야옹야옹");  
    }
}
```

- 상위클래스 Animal에는 템플릿 을 제공하는 play() 메서드와 하위클래스에게 구현을강제하는 cry() 메서드가 존재한다.
- 하위클래스인 Dog와 Cat은 상위클래스인 Animal에서 구현을 강제하는 cry() 메서드를 반드시 구현해야한다.
- 상위클래스 에 공통로직을 수행하는 템플릿메서드와 , 강제하는 추상메서드 , 선택적 오버라이딩이 가능한 훅 메서드를 두는 패턴을 템플릿 메서드패턴이라고한다.


- 구성요소
    - play() : 템플릿 메서드 로직 중 하위클래스에서 오버라이딩한 추상/훅 메서드를 호출한다.
    - cry() : 추상 메서드, 하위클래스가 반드시 오버라이딩해야하낟.
    - doSomething() : 훅 메서드 , 선택적으로 오버라이딩

- "상위클래스의 템플릿 메서드에서 하위클래스가 오버라이딩한 메서드를 호출하는패턴"
- 의존성 역전원척 (DIP) 를 활용하고 있다.
