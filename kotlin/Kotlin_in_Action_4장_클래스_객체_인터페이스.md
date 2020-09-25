# Kotlin in Action

## 4장 클래스, 객체, 인터페이스
- 코틀린의 클래스와 인터페이스는 자바와는 다르다.
- 인터페이스에 프로퍼티 선언이 들어갈 수 있고, 코틀린 선언은 기본적으로 final public 이다.
- 중첩 클래스는 기본적으로 내부클래스가 아니다.
- data 클래스로 선언하면 일부 표준메서드를 생성해준다.

#### 클래스 계층 정의
- 코틀린과 자바의 클래스 정의 방식을 비교해본다.

#### 코틀린 인터페이스
- 코틀린의 인터페이스는 자바8 인터페이스와 유사하다.
- interface 키워드를 사용하며, 차이점 이라면 override 변경자를 반드시 사용해야한다.
```kotlin
/**
 * 자바의 인터페이스와 유사하다.
 * 구현을 가지는 메소드가 존재할 수 있다.
 * 인터페이스 선언시 interface 키워드를 사용한다.
 * override 변경자는 자바의 @Override 애노테이션과 비슷하다.
 * 하지만 자바와 달리 반드시 사용해야한다.
 */
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}

class Button: Clickable {
    override fun click() = println("I was Clicked")
}

fun main(args: Array<String>) {
    Button().click()
}
```

- 둘 이상의 디폴트 구현이 있는경우 구현 클래스에서 반드시 새로운 구현을 제공해야한다.
- super<Type> 을 사용해 특정 상위멤버의 메소드를 호출할 수 있다.

```kotlin
/**
 * 이름과 시그니쳐가 동일한 메소드에 대해 둘 이상의 디폴트 구현이 있는경우
 * 인터페이스 구현체인 클래스에서 명시적으로 새로운 구현을 제공해야한다.
 *
 * super<Type> 을 지정하면 특정 상위타입의 멤버 메소드를 호출할 수 있다.
 */
interface Focusable {
    fun setFocus(b: Boolean) =
            println("I ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm focusable!")
}

class Button2: Clickable, Focusable {
    override fun click() = println("I was clicked")

    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

- 코틀린은 자바 6과 호환되게 설계되어 있다.
- 인터페이스의 디폴트 메소드를 지원하지 않는다.
- 코틀린은 디폴트 메소드가 있는 인터페이스를 일반 인터페이스와 디폴트 메소드 구현이 있는 정적 메소드를 가지는 클래스를 조합해 구현했다.

> 자바에서는 코틀린의 디폴트 메소드 구현에 의존할 수 없다.

#### open, final, abstract 변경자
- 자바에서는 final 키워드를 사용해 상속을 금지한다.
- 하지만 기본적을 상속이 가능하기 때문에 **취약한 기반 클래스 (fragile base class)** 문제가 발생한다.
- 이는 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스를 변경함으로써 깨져버린 경우를 맗나다.
- Effective Java 에서는 상속을 위한 설계와 문서를 갖추거나, 상속을 금지하라 라는 조언을 한다.
- 코틀린도 이 철학을 따르며, 코틀린의 클래스와 메소드는 기본적으로 final 이다.
- 상속을 허용하려면 open 변경자를 사용해야한다.

```kotlin
/**
 * 상속을 허용하기 위해선 open 키워드를 사용해야 한다.
 * 기반 클래스나 인터페이스의 멤버를 오버라이드 하는경우 해당 메소드는 기본적으로 open 이다.
 * 구현을 금지하려면 명시적으로 final 을 붙여 주어야한다.
 */
open class RichButton: Clickable {
    fun disable () {}
    open fun animate () {}
    final override fun click() {}
}
```

`열린 클래스와 스마트 캐스트`
- 기본을 final 로 얻을수 있는 큰 이익은 다양한 경우에 스마트 캐스트가 가능하다는 점이다.
- 때문에 대부분의 프로퍼티를 스마트 캐스트에 활용할 수 있으며 코드를 더 이해하기 쉽게 만든다.

