# Template Callbak Pattern 
- 템플릿 콜백 패턴 은 전략패턴의 변힝이다.
- 스프링 3대 프로그래밍 모델중 DI에서 사용하는 특별한 형태의 전략패턴이다.
- 전략패턴과 모든것이 동일하지만 , 익명 내부 클래스로 정의하여 사용한다는 특징이 있다.

```java
public interface Weapon {
    void attach();
}

public class Character {
    void doFight(Weapon weapon) {
        System.out.println("전투시작 ");
        weapon.attach();
        System.out.println("전투종료 ");
    }
}

public class Main {
    public static void main(String[] args) {
        Character char = new Character();
        char.doFight(new Weapon() {
            @Override
            public void attach() {
                System.out.println("탕탕탕 ~ ");
            }
        });
    }
}
```

- 스프링은 이러한 형식으로 템플릿 콜백패턴을 DI에 적극 활용하고있다.
- 전략패턴 , 템플릿 콜백패턴 등을 잘 기억해두자.
- "전략을 익명 내부 클래스로 구현한 전략 패턴"
