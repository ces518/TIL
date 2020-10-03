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

`추상 클래스`
- 코틀린에서도 자바와 마찬가지로 abstract 키워드를 사용해 추상 클래스를 선언할 수 있다.
- 하지만 인스턴스화 할 수는 없다.
- 추상 멤버는 항상 열려 있기 때문에 추상 멤버 앞에는 open 변경자를 명시할 필요가 없다.

```kotlin
/**
 * 추상 클래스는 기본적으로 open 이기 떄문에 명시할 필요가 없다.
 * 비 추상함수는 기본적으로 파이널 이지만 원한다면 open 으로 오버라이드 할 수 있다.
 */
abstract class Animated {
    abstract fun animate()

    open fun stopAnimating() {

    }

    fun animateTwice() {

    }
}
```

| 변경자 | --- | 설명 |
|---|---|---|
| final | 오버라이드 할 수없음 | 클래스 멤버의 기본 변경자 |
| open | 오버라이드 가능 | 반드시 open 을 명시해야 오버라이드 가능 |
| abstract | 반드시 오버라이드 해야함 | 추상 클래스 멤버에게만 붙일 수 있음. 추상 멤버는 구현이 있으면 안됨 |
| override | 상위 클래스나 상위 인스턴스 멤버를 오버라이드 중 | 오버라이드 하는 멤버는 기본적으로 open, 금지시 final 명시 필요 |

 
#### 가시성 변경자
- 가시성 변경자는 코드 기반에 있는 선언에 대한 클래스 외부 접근을 제어한다.
- 코틀린 가시성 변경자는 자바와 비슷하다.
- 기본 가시성은 자바와 다르게 아무 변경자도 없는 경우 **public** 이다.
- 자바 기본 가시성인 패키지 전용은 코틀린에 존재하지 않는다.
- 코틀린은 패키지를 네임스페이스 관리 용도로만 사용한다.
- 패키지 전용 가시성은 **internal** 이라는 가시성 변경자를 대안으로 제시한다.
- 또한 코틀린은 최상위 선언에 private 을 허용한다.
    - 클래스, 함수, 프로퍼티 등이 포함
    - 해당 선언이 들어있는 파일 내부에서만 사용할 수 있다.
    
| 변경자 | 클래스 멤버 | 최상위 선언 |
|---|---|---|
| public (기본) | 모든 곳에서 볼 수 있음 | 모든 곳에서 볼 수 있음 |
| internal | 같은 모듈 내에서만 볼 수 있음 | 같은 모듈 내에서만 볼 수 있음 |
| protected | 하위 클래스 안에서만 볼 수 있음 | 최상위 선언에 적용 불가 |
| private | 같은 클래스 안에서만 볼 수 있음 | 같은 파일 안에서만 볼 수 있음 |

> 코틀린에서 protected 멤버는 오직 어떤 클래스나 해당 클래스를 상속한 클래스 내에서만 보인다.
> 그 클래스를 확장한 함수는 private, protected 멤버에 접근할 수 없다.

`코틀린 가시성 변경자와 자바`
- 코틀린의 public, protected, private 변경자는 컴파일된 자바 바이트코드 내에서도 그대로 유지된다.
- private 클래스는 패키지-전용 클래스로 컴파일한다.
- internal 변경자는 적절한 가시성이 없다.
- 이는 public으로 컴파일되어, 자바에서는 접근이 가능한 상태가 된다.
- 기술적으로는 internal 멤버의 이름을 보기 힘들기 바꾼다.
    - 기술적으로는 사용할 수는 있지만, 내부 메소드 오버라이딩 방지 및 internal 클래스를 외부 모듈에서 사용하는것을 막기 위함
    
#### 내부 클래스와 중첩된 클래스
- 코틀린의 중첩 클래스는 명시적으로 요청하지 않는 한 바깥 클래스의 인스턴스에 대한 접근 권한이 없다.

```kotlin
import java.io.Serializable

/**
 * 코틀린의 중첩 클래스에 아무런 변경자가 없다면, 자바의 static 중첩클래스와 동일하다.
 * 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하고 싶다면 inner 변경자를 붙여야 한다.
 * 내부 클래스에서 바깥 클래스에 참조하려면 this@<외부클래스명> 형태로 사용해야한다.
 */
interface State: Serializable
interface View {
    fun getCurrentState(): State
    fun restoreState(state: State) {}
}

class Button: View {
    override fun getCurrentState(): State = ButtonState()
    override fun restoreState(state: State) {

    }

    class ButtonState: State {

    }
}

class Outer{
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```

#### 봉인된 클래스 - 계층 정의시 계층 확장 제한
```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun evalKoltinUseWhen(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.right)
        else -> throw IllegalArgumentException("Unknown Expression")
    }

```

- 2장에서 선보인 예제를 살펴보면 문제점이 있다.
- when 을 사용해 타입값을 검사할때 반드시 디폴트 분기인 else 를 강제한다.
- 또한 하위 클래스를 추가하더라도 when 이 모든 경우를 처리하는지 알 수 없다.
- 이에 대한 해법은 **sealed** 클래스이다.
- 상위 클래스에 sealed 변경자를 사용하면 이를 상속한 하위 클래스는 반드시 상위 클래스 안에 중첩시켜야 한다.

```kotlin
/**
 * Sealed 클래스를 사용하면 이를 상속받는 클래스는 반드시 상위 클래스의 내부에 존재해야 한다.
 * sealed 클래스는 자동으로 open 이다.
 * sealed 클래스의 when 분기문에서 디폴트 분기를 사용하지 않으면, 새롭게 추가된 하위클래스에 대한 처리가 없을경우 컴파일러가 알려준다.
 */
sealed class Expr {
    class Num(val value: Int): Expr()
    class Sum(val left: Expr, val right: Expr): Expr()
}

fun eval(e: Expr): Int =
    when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }
```

#### 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언
- 코틀린은 주 생성자와 부 생성자를 구분한다.
- 또한 초기화 블록을 통해 초기화 로직을 추가할 수 있다.

#### 클래스 초기화 - 주 생성자와 초기화 블록
- 클래스의 모든 선언은 중괄호 사이에 들어간다.
- 클래스 명 뒤에 오는 괄호로 쌓인 코드를 **주 생성자 (primary constructor)** 라고 부른다.
- 주 생성자는 생성자 파라미터를 지정하고, 그에 의해 초기화되는 프로퍼티를 정의하는 목적으로 쓰인다. 

```kotlin
/**
 * 클래스 명 뒤에오는 () 괄호로 둘러쌓인 코드를 주 생성자 라고 한다.
 *
 */
class SimpleUser(val nickname: String)

/**
 * 주 생성자를 풀어서 쓰면 다음과 같다.
 */
class User(_nickname: String) { // 파라메터가 1개만 있는 주 생성자
    val nickname: String

    init { // 초기화 블록
        nickname = _nickname
    }
}

/**
* 초기화 블록이 없는 선언
*/
class User(_nickname: String) {
    val nickname = _nickname
}

```

> 위 세가지 모두 클래스를 선언하는 방법중 하나이지만 첫번째 방법이 가장 간결하다.

- constructor 키워드는 주 생성자 혹은 부생성자 정의 시작시 사용한다.
- init 키워드는 초기화 블록을 시작한다, 초기화 블록은 인스턴스화 될때 실행될 초기화 코드가 들어가며, 주 생성자와 함께 사용된다.
    - 주 생성자는 제한적이기 떄문에 초기화 블록이 필요하다.
    - 필요시 클래스 내에 여러 초기화 블록 선언이 가능하다.
    
> 프로퍼티 초기화식, 초기화 블록 내에서만 주 생성자의 파라메터를 참조할 수 있다.

- 함수 파라메터와 동일하게 디폴트값 정의가 가능하다.
```kotlin
class User(val nickname: String,
           val isSubscribed: Boolean = true)
```

- 클래스 인스턴스 생성시 new 키워드 없이 생성자를 직접 호출
```kotlin
val june = User("June")
```

> 모든 생성자 파라메터에 디폴트 값을 지정하면 컴파일러가 자동으로 파라메터가 없는 생성자를 만들며, 디폴트값을 사용해 클래스를 초기화 한다.

- 상위 클래스가 있다면 주 생성자에서 상위 클래스의 생성자를 호출해야 할 필요가 있다.
- 상위 클래스 초기화시 상위 클래스 이름 뒤에 괄호를 써 생성자 인자를 넘긴다.

```kotlin
/**
 * 상위 클래스가 존재한다면, 하위 클래스 생성시 해당 상위 클래스 초기화가 필요하다.
 * 상위 클래스를 초기화 하려면 상위 클래스명 뒤에 괄호를 써 생성자 인자를 넘겨야한다.
 */
open class BaseUser(val nickname: String)

class TwitterUser(nickname: String): BaseUser(nickname)
```

> 클래스 정의시 생성자를 정의하지 않는다면, 파라메터가 없는 디폴트 생성자를 만들어준다.

- 클래스를 상속받는 하위 클래스는 반드시 상위 클래스의 생성자를 호출해야 한다.
- 이 규칙때문에 반드시 괄호가 들어가며, 인터페이스는 생성자가 없기 때문에 인터페이스의 구현체는 괄호가 존재하지 않는다.

#### 부생성자 - 상위 클래스를 다른 방식으로 초기화
- 코틀린에서는 생성자가 여럿 있는 경우가 자바보다 훨씬 적다.
- 자바에서 오버로드한 생성자가 필요한 상황 중 상당수는 코틀린의 디폴트 파라메터 값과 이름이 있는 인자 문법을 사용해 해결한다.

> 디폴트 값을 제공하기 위해 부 생성자를 만들지 말라. 파라메터의 디폴트 값을 생성자 시그니처에 명시하라.

- super() 키워드를 통해 상위 클래스의 생성자를 홏풀한다.
- 또한 this() 키워드를 사용해 자신의 다른 생성자를 호출할 수 있다.
```kotlin
import java.util.jar.Attributes
import javax.naming.Context

open class View {
    constructor(ctx: Context) {
        //
    }

    constructor(ctx:Context, attr: Attributes) {
        //
    }
}

/**
 * super 키워드를 통해 상위 클래스의 생성자를 호출하는 부생성자들
 *
 */
class MyButton: View {
    constructor(ctx: Context): super(ctx) {
        // ...
    }

    constructor(ctx: Context, attr: Attributes): super(ctx, attr) {
        // ...
    }
}
```

> 클래스에 주 생성자가 없다면, 모든 부 생성자는 반드시 상위 클래스를 초기화 하거나, 다른 생성자에게 생성을 위임해야 한다.

- 부 생성자가 필요한 이유는 자바 상호 운용성 때문이다.
    - 또는 인스턴스 생성시 마다 생성방법이 여럿 존재할때 

#### 인터페이스에 선언된 프로퍼티 구현
- 코틀린은 인터페이스에 추상 프로퍼티 선언이 가능하다.
- 추상 프로퍼티 뿐 아니라 게터와 세터가 있는 프로퍼티를 선언할 수 있다.
    - 하지만 이를 뒷받침 하는 필드를 참조할 수 없다.
- 아래는 3가지 방법의 프로퍼티 구현 방법이다.
```kotlin
interface AbstractUser {
    val nickname: String
}

/**
 * 주생성자에 존재하는 Property 구현
 */
class PrivateUser(override val nickname: String): AbstractUser

/**
 * 커스텀 Getter 를 통한 Property 구현
 */
class SubscribingUser(val email: String): AbstractUser {
    override val nickname: String
        get () = email.substringBefore('@') // 커스텀 Getter
}

/**
 * Property 초기화식을 통한 구현
 */
class FacebookUser(val accountId: Int): AbstractUser {
    override val nickname = getFacebookName(accountId) // 프로퍼티 초기화식
}

fun getFacebookName(accountId: Int): String {
    return "Hello $accountId"
}
```

#### Getter, Setter 에서 뒷받침 하는 필드 접근
- 어떤 값을 저장하되, 그 값을 변경하거나 읽을 때 마다 정해진 로직을 수행하는 프로퍼티를 생성하는 방법
- 접근자 본문에서 **field** 라는 식별자를 통해 뒷받침하는 필드에 접근할 수 있다.
- getter 는 읽기전용이며, setter는 읽기/쓰기 모두 가능하다.
- 변경 가능 프로퍼티의 게터와 세터중 한쪽만 정의해도 된다.
- 
```kotlin
class ClazzUser(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            print("""
                Address was change for $name:
                "$field" -> "$value".""".trimIndent()) // 뒷받침하는 필드 읽기
            field = value // 뒷받침하는 필드 변경
        }
}

fun main(args: Array<String>) {
    val user = ClazzUser("Ncucu")
    user.address = "Suwon"
    // Address was change for Ncucu:
    // "unspecified" -> "Suwon".
}
```

> 프로퍼티를 사용하는 입장에서는 뒷받침하는 필드의 유무는 상관이 없다.
> 컴파일러는 field 를 사용하는 프로퍼티는 뒷받침하는 필드를 생성해준다.
> 프로퍼티가 val인 경우 게터에 field가 없어야하고, var 인경우 게터 세터 모두 field 가 없어야 한다.

#### 접근자의 가시성 변경
- 접근자의 가시성은 기본적으로 프로퍼티 가시성과 동일하다.
- 필요시 get, set 앞에 가시성 변경자를 추가하여 접근자의 가시성을 변경할 수 있다.
```kotlin
/**
 * setter 의 접근자를 private 으로 지정해 해당 인스턴스 외부에서는 해당값을 변경할수 없다.
 */
class LengthCounter {
    var counter: Int = 0
        private set

    fun addWord(word: String) {
        counter += word.length
    }

}


fun main(args: Array<String>) {
    val lengthCounter = LengthCounter()
    lengthCounter.addWord("Hello")
    println(lengthCounter.counter)
    // 4
}
```

- lateinit 변경자를 널이 될 수 없는 프로퍼티에 지정하면 프로퍼티를 생성자 호출 이후에 초기화 한다.
- 요청이 들어오면 초기화되는 지연 초기화 프로퍼티는 일반적인 위임 프로퍼티의 일종이다.
- 자바 프레임워크와 호환성을 위해 @JvmField 같은 애노테이션을 지원한다.
- 이는 접근자가 없는 public 필드를 노출시켜준다.
- const 변경자를 사용하면 애노테이션을 더 편리하게 다룰 수 있고, 원시 타입 혹은 String 값을 애노테이션 인자로 사용할 수 있다.

#### 컴파일러가 생성한 메서드 - 데이터 클래스와 클래스 위임
- 자바에서는 equals, hashCode, toString 등의 메소드를 기계적으로 구현해야 한다.
- 코틀린은 이런 기계적으로 생성한느 작업을 하지 않아도 되게끔 지원한다.

#### 모든 클래스가 정의해야하는 메소드
- 코틀린에서도 toString, equals, hashCode 등을 오버라이드 할 수 있다.
- 코틀린 에서는 이런 메소드 구현들을 자동으로 생성해 줄 수 있다.

`동등성 연산에서 == 를 사용한다.`
- 코틀린 에서는 == 연산자가 내부적으로 equals를 호출해서 객체를 비교한다.
- 참조 비교를 위해서는 === 연산자를 사용해야하며, 이는 자바의 == 연산자와 동일하게 동작한다.

```kotlin
/**
 * toString 오버라이드
 */
class ClientToString(val name: String, val postalCode: Int) {
    override fun toString(): String {
        return "Client(name='$name', postalCode=$postalCode)"
    }
}

/**
 * equals 오버라이딩
 */
class ClientEquals(val name: String, val postalCode: Int) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (javaClass != other?.javaClass) return false

        other as ClientEquals

        if (name != other.name) return false
        if (postalCode != other.postalCode) return false

        return true
    }
}

/**
 * HashCode 오버라이딩
 */
class ClientHashCode(val name: String, val postalCode: Int) {
    override fun hashCode(): Int {
        var result = name.hashCode()
        result = 31 * result + postalCode
        return result
    }
}
```

- 자바에서는 equals 오버라이드시 반드시 hashCode도 오버라이드 해야한다.
- JVM 언어에서는 equals() 가 true 를 반환하는 두 객체는 반드시 같은 hashCode() 를 반환해야 한다.

#### 데이터 클래스 - 모든 클래스가 정의해야하는 메소드 자동생성
- 어떤 클래스가 데이터를 저장하는 역할만 수행한다면, toString, equals, hashCode 를 반드시 오버라이드 해야한다.
- 코틀린에서는 data 변경자만 클래스 앞에 붙이면 이런 메소드들을 자동으로 생성해준다.
- 이를 데이터 클래스 라고한다.

```kotlin
/**
 * data class 로 정의함으로써 아래 클래스는 오버라이드된 toString, equals, hashCode 메소드가 생성되어 있다.
 * equals 와 hashCode는 주 생성자에 나열된 프로퍼티 들을 고려해 생성된다.
 */
data class Client(val name: String, val postalCode: Int)
```

#### 데이터 클래스와 불변성 - copy() 메소드
- 데이터 클래스의 프로퍼티가 val 일 필요는 없지만, 불변 클래스로 만들기를 권장한다.
- HashMap등 컨테이너에 데이터 클래스 객체를 담는 경우 불변성이 필수적이다.
- 데이터 클래스 인스턴스를 불변 객체로 더 쉽게 활용할 수 있게 copy 메소드를 제공한다.
- 이는 객체르 복사하면서 일부 프로퍼티를 바꿀 수 있다.

```kotlin
/**
 * data class 로 정의함으로써 아래 클래스는 오버라이드된 toString, equals, hashCode 메소드가 생성되어 있다.
 * equals 와 hashCode는 주 생성자에 나열된 프로퍼티 들을 고려해 생성된다.
 */
data class Client(val name: String, val postalCode: Int)

fun main(args: Array<String>) {
    val client = Client("Hello", 10)
    val client2 = client.copy(name = "Hello2", postalCode = 11)

    println("client = $client")
    println("client2 = $client2")
}
```

#### 클래스 위임 - by 키워드 사용
- 대규모 객체지향 시슽메 설계시 시스템의 취약점은 대부분 구현 상속에 의해 발생하낟.
- 하위클래스가 상위 클래스 메소드의 일부를 오버라이드 하면 하위 클래스는 상위 클래스의 세부 구현 사항에 의존하게 된다.
- 코틀린은 기본적으로 클래스를 final 로 하고 open 변경자로 열어둔 클래스만 확장이 가능하다.
- 상속을 하용하지 않는 클래스에 새로운 동작이 필요한경우 데코레이터 패턴을 사용한다.
- 데코레이터의 단점은 위임을 위한 준비 코드가 필요한다는 것이다.
- 이런 위임을 언어가 제공하는 일급 시민 기능으로 지원한다는 것이 코틀린의 장점이다.

```kotlin
/**
 * 직접 구현해야하는 위임코드를 일급시민으로 언어 차원에서 제공하는 by 를 통해 매우 간단하게 구현이 가능하다.
 */
class DelegationgCollection<T>(
    innerList: Collection<T> = ArrayList<T>()
): Collection<T> by innerList {}
```

#### object 키워드 - 클래스 선언과 인스턴스 생성
- 코틀린에서 object 키워드를 다양한 상황에서 사용하지만 모든경우 클래스 정의와 동시에 인스턴스를 생성한다.
- 객체 선언 (object declaration) 은 싱글턴을 정의하는 방법중 하나
- 동반 객체 (companion object) 는 인스턴스 메소드는 아니지만 특정 클래스와 관련된 메소드와 팩토리 메소드를 담을 때 쓰인다.
- 객체 식은 자바의 익명 내부 클래스 대신 쓰인다.

#### 객체 선언 - 싱글턴을 쉽게 만들기
- 종종 싱글턴 객체가 필요한 경우가 있다.
- 자바는 보통 생성자를 private 으로 제한하고 싱글턴 패턴을 직접 구현한다.
- 코틀린은 객체 선언 기능을 통해 싱글턴을 언어레벨에서 지원한다.
- 객체 선언이란 클래스 선언과 해당 클래스에 속한 단일 인스턴스 선언을 합친 선언이다.
- 생성자는 객체 선언에 쓸 수 없다.
- 싱글턴 객체는 선언문에 있는 위치에서 즉시 만들어진다.
```kotlin
/**
 * object 키워드를 통해 객체선언을 할 수 있다.
 * 코틀린은 object 키워드를 이용해 언어레벨에서 싱글턴을 지원한다.
 */
class Person
object Paroll {
    val allEmployees = arrayListOf<Person>()

    fun calcuateSalary() {
        for (person in allEmployees) {
            //...
        }
    }
}

fun main(args: Array<String>) {
    Paroll.calcuateSalary()
    Paroll.allEmployees.add(Person())
}

```

- 클래스 내부에서도 객체를 선언할 수 있다.
- 코틀린 의 객체선언은 컴파일 될 경우 인스턴스 필드의 이름은 INSTANCE 이다.
- <ObjectName>.INSTANCE 형태로 접근하면 된다.
> 구현 내부에 다른 상태가 필요하지 않은 인터페이스 ex) Comparator 등을 구현할 때는 객체 선언 방식이 좋다.

#### 동반 객체 - 팩토리 메소드와 정적 멤버
- 코틀린 클래스 내에는 정적 멤버가 없다.
- 자바 static 키워드를 지원하지 않는대신 패키지 수준의 최상위 함수 (자바 정적 메소드 역할) 와 객체 선언 (정적 필드 역할 등) 을 활용한다.

> 대부분의 경우 최상위 함수를 사용하는 것을 권장한다.
 
- 팩토리 메소드 처럼 클래스 내부 정보에 접근해야 하는 함수가 필요할 경우 중첩된 객체 선언 멤버함수로 정의해야 한다.
- 클래스 내에 정의된 객체 중 하나에 companion 이라는 특별한 키워드를 사용하면 그 클래스의 **동반 객체**로 만들수 있다.

`동반 객체의 특징`
- 동반 객체의 프로퍼티나 메소드에 접근시 동반 객체가 정의된 클래스명을 사용한다.
- 동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근이 가능하다.
- 오직 하나만 존재할 수 있다.

```kotlin
/**
 * 클래스 내부에 정의된 객체 중 하나에 companion 키워드를 사용하면 해당 객체는 동반 객체가 된다.
 * 동반객체를 사용할떄는 동반객체를 가지고 있는 클래스를 사용하듯이 사용하면된다.
 * 동반객체는 오직 하나만 지정될 수 있다.
 */
class A {
    companion object {
        fun bar() {
            println("Companion Object called")
        }
    }
}

fun main(args: Array<String>) {
    A.bar()
    // Companion Object called
}
```

`동반 객체를 활용한 팩토리 메소드`
```kotlin
/**
 * 기존의 경우 일반 사용자와 소셜 유저를 생성하는 2개의 부 생성자가 필요했다.
 * 이를 동반객체를 이용하여 리팩토링
 */
class BasicUser {
    val nickname: String

    constructor(email: String) {
        nickname = email.substringBefore("@")
    }

    constructor(socialId: Int) {
        nickname = getSocialName(socialId)
    }
}

fun getSocialName(socialId: Int): String = "Hello ~ $socialId"


/**
 * 2개의 부생성자가 필요했던 부분을 동반객체를 활용해 팩토리메소드 형태로 리팩토링
 */
class BasicUserCompanionObject private constructor(val nickname: String) {
    companion object {
        fun newSimpleUser(email: String) = BasicUserCompanionObject(email.substringBefore("@"))
        fun newSocialUser(socialId: Int) = BasicUserCompanionObject(getSocialName(socialId))
    }
}

fun main(args: Array<String>) {
    // 기존의 방식
    BasicUser("ncucu.me@kakaocommerce.com")
    BasicUser(1)

    // 팩토리 메소드 사용
    BasicUserCompanionObject.newSimpleUser("ncucu.me@kakaocommerce.com")
    BasicUserCompanionObject.newSocialUser(1)
}
```

#### 동반 객체 - 일반 객체처럼 사용
- 동반 객체는 클래스 내에 정의된 일반 객체이다.
- 동반 객체에 이름을 붙이거나, 인터페이스 상속, 확장 함수와 프로퍼티 정의 등이 가능하다.

`동반 객체에 이름 붙이기`
- 동반 객체에는 이름을 지정할 수 있다.
- 지정한 이름을 사용하여 동일하게 호출이 가능하다.
```kotlin
/**
 * 동반객체에 이름을 지정할경우 해당 이름을 사용하여 호출, 혹은 기존방식대로 호출이 가능하다.
 */
class SimplePerson(val name: String) {
    companion object Loader {
        fun fromName(name: String): SimplePerson = SimplePerson(name)
    }
}

fun main(args: Array<String>) {
    SimplePerson.fromName("Hello")
    SimplePerson.Loader.fromName("Hello2")
}
```

`동반 객체에서 인터페이스 구현`
- 다른 객체 선언과 동일하게 인터페이스 구현이 가능하다.

```kotlin
/**
 * 동반 객체에서 인터페이스 구현이 가능하다.
 */
interface UserFactory<T> {
    fun create(name: String): T
}

class MyUser(val name: String) {
    companion object: UserFactory<MyUser> {
        override fun create(name: String): MyUser = MyUser(name)
    }
}

fun main(args: Array<String>) {
    MyUser.create("ncucu")
}
```

`동반 객체의 확장 함수`
- 동반 객체도 일반 객체처럼 확장함수 정의가 가능하다.

```kotlin
/**
 * 동반 객체에 대해 확장 함수 정의도 가능하다.
 */
class SimpleA {
    companion object {

    }
}

fun SimpleA.Companion.hello() = println("Hello")

fun main(args: Array<String>) {
    SimpleA.hello()
}
```

#### 코틀린 동반객체와 자바 정적 멤버
- 코틀린의 동반객체에 이름을 지정하지 않았다면, Companion 이라는 이름으로 접근이 가능하다.
    - <RootClass>.Companion
- 이름을 지정했다면 Companion 대신 해당 이름이 쓰이게 된다.
- @JvmStatic 애노테이션을 사용하면 코틀린 클래스의 멤버를 정적 멤버로 만들수 있다.
- 또한 정적 필드가 필요할 경우 @JvmField 애노테이션을 프로퍼티 앞에 사용하여 만들 수 있다.

> 이는 모두 자바 상호운용성을 위해 존재하는 것들이다. 

#### 객체식 - 무명 내부 클래스를 다른 방식으로 작성
- 익명 객체를 정의할 때도 object 키워드를 사용한다.
- 익명 객체는 자바의 익명 내부 클래스를 대신한다.

```kotlin
import java.awt.event.MouseAdapter
import java.awt.event.MouseEvent

/**
 * 무명 객체는 자바의 익명 내부클래스에 대응한다.
 * 객체 선언과 달리 무명 객체는 싱글턴이 아니다.
 */
window.addMouseListener {
    object: MouseAdapter() {
        override fun mouseClicked(e: MouseEvent?) {
            super.mouseClicked(e)
        }

        override fun mouseEntered(e: MouseEvent?) {
            super.mouseEntered(e)
        }
    }
}
```

`SAM`
- 추상 메소드가 하나만있는 인터페이스 (Single Abstract Method) 라는 뜻
- 함수형 인터페이스라고도 한다.
- 자바 람다를 SAM 인터페이스를 구현하는 무명 클래스 대신 사용할 수 있다.

> 자바와 달리 final 이 아닌 변수도 사용할 수 있다.

#### 정리
- 코틀린의 인터페이스는 자바 인터페이스와 비슷하다.
    - 디폴트 구현을 포험하고 프로퍼티를 포함한다.
- 모든 코틀린의 선언은 public final 이다.
- open 키워드를 사용하면 상속과 오버라이딩이 가능해진다.
- internal 선언은 같은 모듈 내에서만 볼 수 있다.
- 중첩 클래스는 기본적으로 내부 클래스가 아니다.
    - inner 키워드를 사용해야지만, 외부 클래스에 대한 참조를 가질 수 있다.
- sealed 클래스를 상속하는 클래스를 정의하려면, 반드시 부모 클래스정의 내에 존재해야한다.
- 초기화 블록과 부 생성자를 통해 인스턴스를 더 유연하게 초기화할 수 있다.
- field 식별자를 통해 프로퍼티 접근자 (게터, 세터) 내에서 프로퍼티 데이터 저장시 쓰이는 뒷받침하는 필드를 참조 가능하다.
- 데이터 클래스를 사용하면 컴파일러가 필수 메소드들을 자동으로 생성해 준다.
- 클래스 위임을 사용하면 위임 패턴을 구현할 때 필요한 수많은 준비코드를 줄일 수 있다.
- 객체 선언을 사용하면 코틀린스럽게 싱글턴 클래스 정의가 가능하다.
- 동반 객체는 자바의 정적 메소드와 필드 정의를 대신한다.
- 동반 객체도 다른 객체와 마찬가지로 인터페이스를 구현할 수 있다. 외부에서 동반객체에 대한 확장함수와 프로퍼티 정의가 가능하다.
- 코틀린의 객체식은 자바의 익명 내부 클래스를 대신한다.
    - final변수를 참조하는등 자바보다 많은 기능을 제공한다.