# Factory Method pattern
- 팩토리는 공장을 의미한다. 객체지향에서 팩토리는 객체를 생성한다.
- 객체를 생성 반환하는 메서드를 의미한다.

```java
public abstract class Animal {
    // 추상 팩토리 메서드
    abstract AnimalToy getToy();
}
// 팩토리 메서드가 생성할 객체의 상위클래스
public abstract class AnimalToy {
    abstract void identify();
}

public class Dog extends Animal {
    // 팩토리 메서드 오버라이딩 
    @Override
    AnimalToy getToy() {
        return new DogToy();
    }
}
// 팩토리 메서드가 생성할 객체
public class DogToy extends AnimalToy {
    public void identify() {
        //구현부..
    }
}

public class Main {
    public static void main(String[] args) {
        // 팩토리 메서드 객체 생성
        Animal dog = new Dog();

        // 팩토리 메서드를 사용하여 객체 생성
        AnimalToy toy = dog.getToye();

        // 팩토리 메서드로 생성한 객체 사용
        toy.identify();
    }
}
```

- 팩토리 메서드 패턴을 한줄로 표현하면 다음과 같다.
- 오버라이드 된 메서드가 객체를 반환하는 패턴
- 의존성 역전의 법칙 DIP가 적용된것을 알 수 있다.
