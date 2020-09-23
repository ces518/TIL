# Kotlin in Action

## 2장 코틀린 기초

#### 함수와 변수
```kotlin
fun main (args: Array<String>) {
    println("Hello World")
}
```

- 함수 선언시 fun 키워드를 사용한다.
- 파라미터 명 뒤에 해당 파라미터의 타입을 쓴다
ex) args: Array<String>
- 함수를 최상위 수준에 정의할 수 있다.
- 배열도 일반클래스와 동일하게 취급한다. 자바와 달리 배열처리 문법이 따로 존재하지 않는다.
- System.out.println 대신 println 을 사용한다.
코틀린 표준 라이브러리는 자바 표준 라이브러리를 간결하게 사용가능하게 감싼 wrapper 를 제공한다.
- 문장 마지막에 세미콜론을 붙이지 않아도 된다.

##### 코틀린 함수의 기본 구조
```kotlin
// 함수명  파라미터 목록  반환타입
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b // 함수 본문
}
```

- 코틀린의 if 는 문장이 아닌 결과를 만드는 식 (expression) 이다.

##### 문(statement)과 식(expression)의 구분
- 코틀린 에서 if는 문이 아니며, 식이다.
- **식**은 값을 만들어 내어 다른 식의 하위 요소로 계산에 참여가 가능하다.
- **문**은 자신을 둘러싼 가장 안쪽 블록의 최상위 요소이며 아무런 값을 만들어내지 않는다.

> 자바에서는 모든 제어 구조가 문 이지만, 코틀린 에서는 루프를 제외한 대부분의 제어 구조가 식이다.

> 대입 문은 자바에서 식이었으나, 코틀린에서는 문이 되었다.

##### 식이 본문인 함수
- 앞선 함수를 중괄호를 없애고 다음과 같이 간결하게 표현할 수 있다.
```kotlin
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

> 본문이 중괄호로 쌓인 함수를 **블록이 본문인 함수** 라고 하고, 등호와 식으로 이루어진 함수를 **식이 본문인 함수** 라고 표현한다.

- 코틀린에서는 식이 본문인 함수가 자주 쓰이며, 반환 타입을 생략하여 좀 더 간결하게 표현할 수 있다.
```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

> 반환 타입을 생략할 수 있는 이유는 코틀린 컴파일러의 **타입 추론** 기능 때문이다.

> 유의할 점은 **식이 본문인 함수** 만 반환 타입 생략이 가능하다.

#### 변수
- 자바에서 변수 선언시에는 타입이 가장 앞에 위치하지만, 코틀린에서는 이를 생략하는 경우가 많다.
```kotlin
/**
 * 타입으로 변수 선언을 시작하면 식과 변수선언을 구분할 수 없다.
 * 때문에 코틀린에서는 키워드로 변수 선언을 시작하는 대신 변수 명 뒤에 타입을 명시하거나 생략을 허용한다.
 * 부동소수점 상수를 사용한다면 변수 타입은 Double 이 된다.
 * 초기화 식을 사용하지 않고 변수를 선언하려면 변수 타입을 반드시 명시해야 한다.
 */
fun main (args: Array<String>) {
    val question = "삶, 우주, 그리고 모든 것에 대한 궁극적인 질문"

    val answer = 42

    val typedAnswer: Int = 42

    val yearsToCompute = 7.5e6 // Double Type

    val notInitializeVariable: Int
    notInitializeVariable = 42
}
```

##### 변수 선언 키워드
- 변수 선언시 val, var 두가지 키워드를 사용한다.
- val: 변경 불가능한 참조를 저장하는 변수이다.
    - 자바 final 변수에 해당한다.
- var: 변경 가능한 참조를 저장하는 변수이다.
    - 변수의 값을 변경할수는 있지만 **변수의 타입은 바뀌지 않는다.**

> 모든 변수를 val 키워드로 사용해 선언하고, 필요한 경우에만 var 로 변경할 것을 권장한다.


##### 문자열 템플릿
```kotlin
/**
 * 문자열 템플릿은 자바의 문자열 접합 연산 ("Hello," + name + "!") 와 동일한 기능이지만, 좀 더 간결하다.
 * 단순히 변수명으로만 한정되지 않으며, 복잡한 식은 중괄호 ( { } ) 를 사용하여 문자열 템플릿에 사용할 수 있다.
 */
fun main (args: Array<String>) {
    val name = if (args.size > 0) args[0] else "Kotlin"
    println("Hello, $name")
    // println("Hello, ${args[0]}")
}
```
- 한글을 문자열 템플릿에서 사용시에는 주의 해야한다.
- 변수명 바로 뒤에 한글을 붙여서 사용하면 컴파일러가 영문자와 한글을 한꺼번에 식별자로 인식해서 unresolved reference 오류를 발생시킨다.
- 해결 방법은 변수 명을 중괄호로 감싸는 것이다.
- 변수 명만 사용하더라도 중괄호를 감싸는 습관을 들이면 좋다.

#### 클래스와 프로퍼티

##### 클래스
- 코틀린에서 클래스를 선언하는 간단한 구문을 소개한다.
```java
public class Person {
    private final String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}
```
- 위 자바 빈 클래스를 kotlin 으로 변환하면 다음과 같다.
```kotlin
class Person(val name: String)
```
> 이런 유형의 클래스를 값 객체 (value object) 라고 한다.
- 자바 -> 코틀린을 변경했을때 public 접근 제어자가 생략되었다. 코틀린의 기본 가시성은 public 이므로 이런 경우 생략해도 된다.

##### 프로퍼티
- 클래스 개념의 목적은 데이터를 캡슐화 하는 것이다.
- 자바에서는 일반적으로 멤버 필드의 가시성을 private 으로 하고, 접근 제어 메소드를 제공한다.
- 위 두가지를 묶어 프로퍼티라고 부른다.
- 코틀린 프로퍼티는 위 두가지를 완전히 대체하며, val 또는 var 를 이용해 선언한다.

```kotlin
/**
 * 코틀린 프로퍼티를 정의할 때는 변수 선언과 마찬가지로 val 혹은 var 키워드를 사용한다.
 * val 키워드를 사용하면, 읽기 전용 프로퍼티가 생성되며, 비공개 필드와 getter 를 생성한다.
 * var 키워드를 사용하면 쓰기 가능한 프로퍼티로, 비공개 필드, getter, setter 를 생성한다.
 */
class Person(
    val name: String,
    var isMarried: Boolean
)
```

> 코틀린에서 프로퍼티를 선언하는 방식은 프로퍼티와 관련 있는 접근자를 선언하는 것이다.

##### 프로퍼티 사용
```java
Person person = new Person("Bob", true);
person.getName();
person.isMarried();
```

```kotlin
val person = Person("Bob", true)
person.name
person.isMarried
```

- 자바와 코틀린의 차이는 코틀린은 **프로퍼티를 직접 사용** 했다는 점이다.
- 프로퍼티를 직접 사용해도 코틀린이 자동으로 게터를 호출해준다. (로직이 간결해진다)
- 세터도 마찬가지로 동작한다.

> 자바에서 작성된 프로퍼티도 코틀린에서 동일하게 사용이 가능하다.

##### 커스텀 프로퍼티
- 커스텀 프로퍼티는 값을 저장하는 필드가 없고, 프로퍼티 접근시 마다 getter 가 값을 매번 다시 계산한다.
```kotlin
/**
 * 코틀린 에서는 커스텀 프로퍼티 기능을 제공한다.
 * 자체 구현을 제공하는 게터만 존재하는 프로퍼티이다.
 * 값을 저장하는 필드가 없고, 프로퍼티에 접근할 때 마다 getter 가 매번 다시 계산한다.
 */
class Rectangle(val height: Int, val width: Int) {
//    val isSquare: Boolean
//        get() {
//            return height == width
//        }
    val isSquare: Boolean
          get() = height == width
}
```

#### 디렉터리와 패키지
- 코틀린에도 자바와 비슷한 패키지 개념이 있다.
- 모든 코틀린 파일의 package 문을 넣을수 있으며, 자바와 동일하게 동작한다. 
```kotlin
package geometry.shapes

import java.util.Random

/**
 * 코틀린에서는 자바와 동일하게 package 개념이 존재한다.
 * 자바와 동일하게 동작한다.
 */
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() = height == width
}

fun createRandomRetangle(): Rectangle {
    val random = Random()
    return Rectangle(random.nextInt(), random.nextInt())
}
```

> 코틀린에서는 여러 클래스를 한 파일에 넣을 수 있고, 파일의 이름도 마음대로 정할 수 있다.
> 디스크상의 어느 디렉터리에 소스코드 파일을 취시키든 관계 없다.

대부분의 경우 자바와 동일하게 패키지별로 디렉터리를 구성하는 편이 낫다.

#### enum 과 when

##### enum 클래스
- 코틀린에서 switch는 when이며, 자바의 switch 를 대체하되 훨씬 더 강력한 기능을 제공한다.
- enum 키워드와 class 키워드를 사용해서 enum 을 정의한다.
- enum 키워드는 소프트 키워드 (soft keyword) 라고 불리며, 다른곳에서는 이름으로 사용할수 있다.
- 자바와 동일하게 프로퍼티나 메소드도 정의할 수 있다.

```kotlin
/**
 * enum 은 자바선언보다 코틀린 선언에 키워드가 더 많은 흔치 않은 예 이다.
 * 코틀린 에서의 enum 은 소프트 키워드 라 불린다.
 * enum 은 class 앞에 존재할 때만 특별한 의미를 가지지만 다른 곳에서는 이름에 사용이 가능하다.
 *
 * 코틀린 enum 에서는 enum 목록과 메서드 정의 사이에 세미콜론을 넣어 구분한다.
 * 코틀린에서 유일하게 세미콜론을 사용하는 부분이다.
 */
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}

enum class ColorWithPropertyAndMethod (
    val r: Int,
    val g: Int,
    val b: Int
) {
    RED(255, 0, 0),
    ORANGE(255, 165, 0),
    YELLOW(255, 255, 0);

    fun rgb() = (r * 256 + g) * 256 + b
}
```

##### when 으로 enum 다루기
- 코틀린의 when 도 자바의 switch 와 마찬가지로 enum 을 다룰 수 있다.
- if 와 마찬가지로 when은 값을 만들어 내는 식이며, 식이 본문인 함수에 when 을 바로 사용할 수 있다.

```kotlin
/**
 * 코틀린 에서의 switch는 when 이다.
 * if 와 마찬가지로 값을 만들어내는 식이다.
 * 자바와 달리 break 을 넣지 않아도 된다.
 * 한 분기내에서 여러 값을 매치 패턴으로 사용할 경우 값 사이를 콤마(,) 로 구분한다.
 *
 * 아래 예제는 when 을 본문으로 하는 (식이 본문인 함수) 이다.
 */
fun getMneomonic(color: Color) =
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO, Color.VIOLET -> "In"
    }

fun main(args: Array<String>) {
    println(getMneomonic(Color.RED))
}
```

##### when 과 임의의 객체를 함께 사용하기
- 코틀린의 when 은 분기 조건에 임의의 객체 사용이 가능하다.
- 1.3 버전 부터는 when 의 조건에 해당하는 대상을 변수에 담아 사용이 가능하다.

```kotlin
/**
 * 코틀린의 when은 자바의 switch 보다 강력한 기능을 제공한다.
 * 분기 조건에 임의의 객체를 허용한다.
 *
 * setOf() 함수는 코틀린 표준 라이브러리에서 제공하는 함수로, 인자로 전달되는 객체를 원소로 가지는 Set객체 로 만들어 준다.
 * 분기조건에 있는 객체를 매치할 때 동등성 (equality)을 사용한다.
 *
 * 코틀린 1.3 버전 부터는 when의 검사 대상을 변수에 담아 새로운 명칭으로 사용이 가능하다.
 */
fun max(c1: Color, c2: Color) =
    when (setOf(c1, c2)) {
        setOf(Color.RED, Color.YELLOW) -> Color.ORANGE
        setOf(Color.YELLOW, Color.BLUE) -> Color.GREEN
        setOf(Color.BLUE, Color.VIOLET) -> Color.INDIGO
        else -> throw Exception("Dirty Color")
    }
```

##### 인자가 없는 when
- 위의 예제코드는 호출이 빈번할 경우 비효율적이게 된다.
- 매 비교마다 새로운 Set 객체를 생성하기 때문에 잦은 GC가 예상된다.

```kotlin
/**
 * 위 함수가 빈번하게 호출된다면 비효율적이게 된다.
 * 매번 비교때 마다 새로운 Set 객체를 생성하기 때문에 호출빈도가 높다면 GC 가 자주 일어나게 되므로 아래와 같이 최적화 해야할 수도 있다.
 * 하지만 아래 방법은 성능적으론 이득이지만, 가독성은 더 떨어진다는 문제가 있다.
 */
fun mixOptimized(c1: Color, c2: Color) =
    when {
        (c1 == Color.RED && c2 == Color.YELLOW) ||
        (c1 == Color.YELLOW && c2 == Color.RED) ->
            Color.ORANGE
        (c1 == Color.YELLOW && c2 == Color.BLUE) ||
        (c1 == Color.BLUE && c2 == Color.YELLOW) ->
            Color.GREEN
        (c1 == Color.BLUE && c2 == Color.VIOLET) ||
        (c1 == Color.VIOLET && c2 == Color.BLUE) ->
            Color.INDIGO
        else -> throw Exception("Dirty Color")
    }

```

##### 스마트 캐스트 - 타입 검사와 타입 캐스트 조합
- 간단한 산술식을 계산하는 함수를 생성하는 예제
- 함수가 받을 산술식은 오직 두 수를 더하는 연산만 가능하게 한다.

```kotlin
/**
 * 식을 트리구조로 저장한다.
 * Num 객체는 항상 리프 노드이며, Sum 은 자식이 둘 있는 중간 노드이다.
 * 식을 위한 Expr 인터페이스를 선언하며, 공통 타입 역할을 위한 마커 인터페이스로 사용한다.
 *
 * (1 + 2) + 4 라는 식을 저장한다면, Sum(Sum(Num(1), Num(2)), Num(4)) 라는 구조의 객체가 생성된다.
 * 위 식의 결과 값은 7이여야만 한다.
 *
 */
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```

- Expr 인터페이스에는 두 가지 구현클래스가 존재하며, 두가지 경우를 고려해야 한다.
1. 어떤 식이 수라면 그 값을 반환
2. 어떤 식이 합계라면 좌항과 우항의 값을 계산한 뒤 합한 값을 반환

`자바 스타일로 만든 함수`
```kotlin

/**
 * 자바 스타일로 만든 함수
 * 자바에서는 instanceof 키워드를 사용해 변수타입을 검사한다.
 * 코틀린에서는 is 키워드를 사용해 변수 타입을 검사한다.
 * 자바에서는 명시적으로 변수 타입을 캐스팅해주어야 한다.
 * 하지만 코틀린에서는 is 키워드로 타입 검사를 하고나면 컴파일러가 대신 캐스팅을 해준다.
 * 이를 스마트 캐스팅 이라고 한다.
 * 주의점은 그 값이 바뀔수 없는 경우에만 동작한다.
 * ex) val이 아니거나 val이지만 커스텀 접근자를 사용할 경우 항상 같은 값을 반환함을 보장하지 않기 때문
 * 원하는 타입으로 명시적으로 캐스팅 할때는 as 키워드를 사용한다.
 */
fun eval (e: Expr): Int {
    if (e is Num) {
        val n = e as Num
        return n.value
    }
    if (e is Sum) {
        return eval(e.right) + eval(e.left)
    }
    throw IllegalArgumentException("Unknown expression")
}
```

`코틀린 스타일로 만든 함수`
```kotlin
/**
 * 코틀린 스타일의 if 를 사용해서 리팩토링
 * 식이 본문인 함수와 스마트 캐스트를 활용함
 */
fun evalKotlinUseIf(e: Expr): Int =
    if (e is Num) {
        e.value
    } else if (e is Sum) {
        eval(e.right) + eval(e.left)
    } else {
        throw IllegalArgumentException("Unknown expression")
    }
    
/**
 * 위 코드를 when 을 사용해서 리팩토링
 */
fun evalKoltinUseWhen(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.right)
        else -> throw IllegalArgumentException("Unknown Expression")
    }

/**
 * if, when 모두 분기에 블록을 사용할수 있다.
 * 블록을 사용할 경우 블록 내 맨 마지막 식이 결과값이 된다.
 * 블록의 마지막 식이 결과 라는 규칙은 블록이 값을 만들어내야 하는 경우 항상 성립한다.
 */
fun evalWithLogging(e: Expr): Int =
    when (e) {
        is Num -> {
            println("num: ${e.value}")
            e.value
        }
        is Sum -> {
            val left = eval(e.right)
            val right = eval(e.right)
            println("sum: $left + $right")
            left + right
        }
        else -> throw IllegalArgumentException("Unknown Expression")
    }
```

#### while 과 for 루프
- 코틀린 특성 중 자바와 가장 비슷한 것이 이터레이션
- while 은 자바와 동일한 형태이고, for는 자바의 for-each 루프에 해당하는 형태만 존재한다.

##### while
- while 과 do-while 루프가 존재하며, 자바와 다르지 않다.

```kotlin
/**
 * 코틀린에는 while, do-while이 있으며 자바와 동일한 기능을 제공한다..
 */
fun main(args: Array<String>) {
    while (true) {
        /*...*/
    }

    do {
        /*...*/
    } while (true)
}
```

##### 수에 대한 이터레이션 - 범위와 수열
- 코틀린에는 자바의 for 문에 해당하는 요소가 없다.
- 이를 대신 하기 위해 코틀린에서는 범위 (range) 를 사용한다.
- 범위는 기본적으로 두 값으로 이뤄진 구간이며, 두 값을 숫자 타입이고 .. 연산자를 이용해 범위를 만든다.
- 코틀린의 범위는 폐구간 또는 양끝을 포함한다.
- 어떤 범위에 속하는 값을 일정 순서로 이터레이션 하는 것을 수열 이라고 한다.

```kotlin
/**
 * 1 ~ 10 까지의 범위 (range) 를 만듦
 */
val oneToTen = 1..0

/**
 * when 을 사용한 fizzBuzz 구현
 */
fun fizzBuzz(i: Int) = when {
    i % 15 == 0 -> "FizzBuzz"
    i % 3 == 0 -> "Fizz"
    i % 5 == 0 -> "Buzz"
    else -> "$i "
}

/**
 * range 의 마지막 수를 제외하고 싶다면 until 키워드를 사용하라.
 */
fun main(args: Array<String>) {
    for (i in 1..100) {
        print(fizzBuzz(i))
    }

    // 100 부터 거꾸로 세며, 2씩 감소한다.
    for (i in 100 downTo  1 step 2) {
        print(fizzBuzz(i))
    }

    for (i in 100 until 100) {

    }
}

```

##### Map Iteration
- map 을 사용해 이터레이션 할때 컬렉션의 원소를 풀어서 두 변수로 사용할 수 있다.
- 이를 구조분해 구문이라고 하며, 이는 맵이 아닌 컬렉션에서도 사용이 가능하다.

```kotlin
import java.util.*

/**
 * for 루프를 사용해 이터레이션 하려는 컬렉션의 원소를 풀어서 letter, binary 두 변수로 사용한다.
 * 또한 맵을 사용할때 get, put 을 사용하는 대신 map[key], map[key] = value 형태로 사용할 수 있다.
 * 구조분해 구문은 맵이 아닌 컬렉션에도 사용할 수 있다.
 */
fun main(args: Array<String>) {
    val binaryReps = TreeMap<Char, String>()

    for (c in 'A'..'F') {
        val binary = Integer.toBinaryString(c.toInt())
        binaryReps[c] = binary // == binaryReps.put(c, binary) 와 동일하다.
    }

    // 구조분해 구문
    for ((letter, binary) in binaryReps) {
        println("$letter = $binary")
    }

    // 현재 원도의 인덱스를 유지하며 이터레이션
    val list = arrayListOf("10", "11", "1001")
    for ((index, element) in list.withIndex()) {
        println("$index: $element")
    }
}
```

##### in 으로 컬렉션이나 범위의 원소 검사
- in 연산자를 이용해 어떤 값이 범위에 속하는지 검사가 가능하며, 반대는 !in 으로 사용한다.

```kotlin
/**
 * in 연산자를 사용해서 컬렉션이나 범위에 원소가 포함되는지 확인이 가능하다.
 */
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'
```

`when 에서 in 키워드 사용`

```kotlin
/**
 * when 에서도 in 연산자를 사용할 수 있다.
 */
fun recognize(c: Char) = when(c) {
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
    else -> "I don't know.."
}
```

> 범위는 문자에만 국한되지 않으며, 비교가 가능한 클래스 (java.lang.Comparable 인터페이스 구현체)일 경우 
> 해당 클래스 인스턴스 객체로 범위를 만들 수 있다.

#### 코틀린의 예외 처리
- 코틀린의 예외 처리는 자바나 다른 언어와 비슷하다.
- 함수 실행중 오류 발생시 예외를 던질 수 있으며, 클라잉너트에서는 그 예외를 잡아 처리할 수 있다.
- 기본 예외 처리 구문은 자바와 비슷하며, new 키워드도 필요없다.
- 코틀린의 throw 는 식이므로, 다른 식에 포함될 수 있다.

```kotlin
import java.lang.IllegalArgumentException

/**
 * 코틀린의 기본 예외처리 구문은  자바와 비슷하다.
 * 차이점 이라면 throw 도 식이기 때문에 다른 식에 포함될 수 있다.
 */
fun main(args: Array<String>) {
    val percentage: Int = 50

    if (percentage !in 0..100) {
        throw IllegalArgumentException("A percentage value must be between 0 and 100: $percentage")
    }
}
```

##### try, catch, finally
- 자바와 마찬가지로 try-catch-finally 구문을 지원한다.
- 자바와 큰 차이는 throws 절이 코드에 없다.
- 자바에서는 함수 작성시 선언 문 뒤에 throw IOException 을 사용해야 한다.
    - IOException 은 체크 예외이기 때문
- 자바에서 체크 예외를 명시적으로 해야하지만 코틀린은 체크예외와 언체크 예외를 구분하지 않는다.

```kotlin
import java.io.BufferedReader
import java.lang.NumberFormatException

/**
 * try-catch-finally 는 자바와 동일하다.
 * 코틀린은 체크예외와 언체크 예외를 구분하지 않는다.
 * 또한 try-with-resource 구문은 지원하지않는데, 이는 라이브러리 함수로 같은 기능을 구현한다.
 */
fun readNumber(reader: BufferedReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    } catch (e: NumberFormatException) {
        return null
    } finally {
        reader.close()
    }
}
```

`try를 식으로 사용하기`
```kotlin

/**
 * try 를 식으로 사용하기
 * try는 if 와 when 과 마찬가지로 식이다.
 * if 와는 달리 try는 본문을 반드시 중괄호로 둘러 써야한다.
 */
fun readNumberV2(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return
    }
    println(number)
}
```
- catch 블록 내에서 return 문을 사용한다.
- 예외 발생시 다음 코드는 실행되지 않게 되는데 게속 진행하고 싶다면 catch블록 에서 null 값을 만들어 내야한다.

#### 정리
- 함수 정의시 fun 키워드 사용
- val는 읽기전용 변수, var 는 변경가능한 변수
- 문자열 템플릿을 사용하면 코드가 간결해 지며 $, ${} 를 사용해 변수나 식의 값을 문자열내에 넣을 수 있음
- 코틀린에서는 값 객체 클래스를 매우 간결하게 표현가능
- if 는 코틀린에서 식이며, 값을 만들어 냄
- when 은 자바보다 기능이 더 강력하다
- 컴파일러가 스마트 캐스트 기능을 지원한다.
- for, while, do-while 기능을 제공하지만 for 는 자바의 for 보다 편리하며, 컬렉션 이터레이션시 빛을 발한다.
- 1..5 와 같이 범위를 만들어 낼 수 있다.
- 예외 처리는 자바와 비슷하며, 체크예외와 언체크 예외를 구분하지 않기 때문에 throws 할 필요가 없다.