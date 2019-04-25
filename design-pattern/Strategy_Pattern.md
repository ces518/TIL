# Strategy Pattern
- 전략패턴은 디자인 패턴의 꽃이다.
- 전략패턴을 구성하는 세 요소는 반드시 기억해두자.


### 전략패턴의 구성요소
    - 전략 메서드를 가진 전략객체
    - 전략 객체를 사용하는 컨텍스트
    - 전략 객체를 생성해 컨텍스트에 주입하는 클라이언트

```java
public interface Weapon {
    void attack();
}

public class Gun implements Weapon {
    @Override
    public void attack() {
        System.out.println("총 빵빵 ~");
    }
}

public class Sword implements Weapon {
    @Override
    public void attack() {
        System.out.println("칼 챙챙 ~");
    }
}

public class Character {
    void doFight(Weapon weapon) {
        System.out.println("전투 준비");
        weapon.attack();
        System.out.println("전투 종료");
    }
}

public class Main {
    public static void main(String[] args) {
        Weapon weapon = null;
        Character char = new Character();

        weapon = new Gun();
        char.doFight(weapon);

        weapon = new Sword();
        char.doFight(weapon);
    }
}
```

- "클라이언트가 전략을 생성해 전략을 실행할 컨텍스트에 주입하는 패턴"
