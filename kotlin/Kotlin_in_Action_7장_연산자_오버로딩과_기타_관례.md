# Kotlin in Action

## 7장 연산자 오버로딩과 기타 관례
- 자바에서는 for..in 루프에서 java.lang.Iterable 구현체를 사용하거나, try-with source 문에 java.lang.AutoCloseable 구현체를 사용할수 있다.
- 이뫄 비슷하게 코틀린에서도 어떤 언어 기능이 사용자 작성된 함수와 연결되는 몇가지가 있다.


#### 산술 연산자 오버로딩
- 코틀린에서 관례를 사용하는 갖아 단순한 예는 산술 연산자 이다.
- BingInteger 클래스를 다룬다면 add 보단 + 연산하는 것이 더 낫고, 컬렉션에 원소를 추가하는 경우 += 이 사용가능하면 좋다.
- 코틀린에서는 이런것들이 가능하다.

#### 이항 산술 연산 오버로딩
- 코틀린에서는 plus 라는 이름의 메소드를 정의하면 해당 인스턴스에 대해 + 연산자를 사용할 수 있다.
- operator 키워드를 반드시 붙여 주어야한다, 만약 operator 키워드가 없다면, operator modifier is required... 예외가 발생한다.

```kotlin
/**
 * 코틀린에서 plus 라는 이름의 메소드를 정의하면 해당 인스턴스에 대해 + 연산자를 이용가능하다.
 * 이 때 반드시 operator 키워드를 붙어주어야 한다.
 */

data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

fun main(args: Array<String>) {
    val p1 = Point(10, 20)
    val p2 = Point(30, 40)
    println(p1 + p2)
}
```

> 멤버 함수뿐 아닌 확장함수를 통해 정의하더라도 동일한 기능을 사용할 수 있다.

`오버로딩 가능한 이항 산술 연산자`

| 식 | 함수 명 |
|---|---|
| a * b | times |
| a / b | div |
| a % b | mod( 1.1 부터 rem)
| a + b | plus |
| a - b | minus |

#### 연산자 함수와 자바
- 코틀린 연산자를 자바에서 호출하는 것은 쉽다.
- 모두 함수로 정의되며 일반 함수로 호출할 수 있다.
- 반대의 경우 함수 명이 코틀린 관례에만 맞아 떨어진다면 연산자 식을 통해 호출할 수 있다.
    - 자바에서는 따로 연산자 표시가 불가능 하므로 operator 키워드를 사용할 수 없다.
    - 메서드 명과 파라메터 수만 문제가 된다.
    
> 코틀린 연산자가 자동으로 교환 법칙 ( a op b == b op a ) 을 지원하지는 않는다.

- 연산자 함수의 반환 타입이 꼭 두 피연산자중 하나와 일치하지 않아도 된다.
- 또한 일반 함수와 동일하게 operator 함수도 오버로딩 할 수 있다.

#### 복합 대입 연산자 오버로딩
- plus 같은 연산자를 오버로딩 할 경우 +=, -= 와 같은 복합 대입 연산자도 함께 지원한다.

```kotlin
/**
 * plus 와 같은 연산자 오버로딩을 했다면, +=, -= 과 같은 복합 연산자에 대한 오버로딩도 지원한다.
 */
fun main(args: Array<String>) {
    val numbers = ArrayList<Int>()
    numbers += 42
    println(numbers[0])
}
```

- 반환 타입이 Unit 인 plusAssign 함수를 정의하면 += 연산자에 해당 함수를 사용한다.
- 다른 복합 대입 연산자도 minusAssign 등의 이름을 사용한다.

> 코틀린 표준 라이브러리는 변경 가능한 컬렉션에 대해 plusAssign 을 정의하고 있다.

* 어떤 클래스가 이 두 함수를 모두 정의하고, += 에 사용가능 한 경우 컴파일러는 오류를 내뱉는다.
- plus, plusAssign 연산을 동일하게 정의하지 않아야한다.

> 코틀린 표준 라이브러리는 +, - 는ㄴ 항상 새로운 컬렉션을 반환하고, +=, -= 연산자는 항상 변경가능한 컬렉션에서 작용해 객체 상태를 변화시킨다.
> 읽기 전용 컬렉션에서 +=, -= 는 변경이 적용된 복사본을 반환한다. (var 로 선언한 컬렉션에만 사용할 수 있음) 

#### 단항 연산자 오버로딩
- 단항 연산자를 오버로딩하는 것도 이항 연산자와 동일하다.
- 차이점 이라면 함수에 인자가 없다는 점이다. (당연한 것..)

```kotlin
/**
 * 단항 연산자를 오버로딩하는 경우도 이항 연산자와 동일하다.
 */
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

fun main(args: Array<String>) {
    val p = Point(10, 20)
    println(-p)
}
```

`오버로딩 가능한 단항 산술 연산자`

| 식 | 함수 명 |
|---|---|
| +a | unaryPlus |
| -a | unaryMinus |
| !a | not |
| ++a, a++ | inc |
| --a, a-- | dec |

#### 비교 연산자 오버로딩
- 모든 객체에 대해 비교 연산을 수행할 수 있다.
- equals 혹은 compareTo 메소드를 호출하지 않고 == 비교 연산자를 지원한다.

#### 동등성 연산자: equals
- 4장에서 동등성을 다루면서, 코틀린은 == 연산자 호출을 equals 메소드 호출로 컴파일 한다는 사실을 알고 있다.
- 하지만 이는 관례를 적용한 것에 불과하다.
- != 연산자를 사용하는 경우에도 equals 로 컴파일된다.
- ==, != 는 내부에서 인자가 널인지 검사하고 아닌 경우에만 equals 를 호출한다.

> a == b 는 a?.equals(b) ?: (b == null) 로 컴파일 된다.

#### 순서 연산자: compareTo
- 자바에서 정렬, 최대값, 최솟값 등을 비교할 때 Comparable 인터페이스를 구현해야 한다.
- compareTo 메소드는 한 객체와 다른 객체의 크기를 비교하여 정수값을 반환한다.
- 코틀린도 같은 인터페이스를 지원하고 compareTo 메소드를 호출하는 관례를 지원한다.
- 비교 연산자 (<, >, <=, >=) 는 compareTo 호출로 컴파일 된다.

```kotlin
class Person (
        val firstName: String,
        val lastName: String
): Comparable<Person> {
    override fun compareTo(other: Person): Int {
        return compareValuesBy(this, other,
                Person::lastName, Person::firstName)
    }
}
```
- 코틀린 표준 라이브러리인 compareValuesBy 함수를 이용해 compareTo 를 간결하게 정의했다.
- 두 객체와 여러 비교 함수를 인자로 받는다.
- 먼저 첫번째 비교 함수에서 두 객체를 비교하고, 두번째 비교 함수를 통해 두 객체를 비교한다.
- 각 비교 함수는 람다 이거나, 프로퍼티, 메소드 참조 일 수 있다.

> 처음에는 성능에 신경 쓰지 말고 이해하기 쉽고 간결하게 코드를 작성한 뒤 호출 빈도가 높아짐에 따라 추후 성능 이슈가 있다면 그때 개선하라.

> 코틀린에서 구현한 Comparable 인터페이스는 자바 쪽의 정렬 메소드 등에서도 사용이 가능하다.

#### 컬렉션과 범위에 대해 쓸 수 있는 관례
- 컬렉션을 다룰 때 가장 많이 쓰는 연산은 인덱스를 사용해 원소를 읽거나 쓰는 연산과
- 어떤 값이 컬렉션에 속해있는지 검사하는 연산이다.
- 인덱스를 이용해 원소를 설정하거나 가져올경우 a[index] 형태로 식을 사용할 수 있는데 이를 **인덱스 연산자** 라고 한다.
- in 연산자는 원소가 컬렉션 이나 범위에 속하는지 검사 혹은 이터레이션할 때 사용한다.

#### 인덱스로 원소에 접근 get, set
- 코틀린에서 대괄호를 이용해 배열이나 맵의 원소에 접근할 수 있다.
- 이를 인덱스 연산자라고 하며, 이를 이용해서 Read 행위는 get 메소드로 변환되고, Write 행위는 set 으로 변환된다.
- Map, MutableMap 인터페이스에는 이미 두 메소드가 존재한다.

```kotlin
import java.lang.IndexOutOfBoundsException

data class MutablePoint(var x: Int, var y: Int);

operator fun MutablePoint.get(index: Int): Int =
    when (index) {
        0 -> x
        1 -> y
        else ->
            throw IndexOutOfBoundsException("Invalid coordinate $index")
    }

operator fun MutablePoint.set(index: Int, value: Int) =
    when (index) {
        0 -> x = value
        1 -> y = value
        else ->
            throw IndexOutOfBoundsException("Invalid coordinate $index")
    }

fun main(args: Array<String>) {
    val p1 = MutablePoint(0, 1)
    println(p1[0]) // 0 으로 접근하면 x 를
    println(p1[1]) // 1로 접근하면 y 를 반환한다.

    p1[0] = 10
    p1[1] = 20

    println(p1)
}
```

> get, set 관례 모두 구현이 가능하다.

#### in 관례
- 컬렉션이 지원하는 다른 연산자는 in 연산자가 있다.
- 객체가 컬렉션에 들어 있는지 검사하고 이에 대응하는 함수는 contains 이다.
- in 의 우항의 객체는 contains 메소드의 수신객체, 좌항은 메소드 인자로 전달된다.
```kotlin
/**
 * contains 함수를 구현하면 in 관계를 사용할 수 있다.
 * 어떤 객체가 컬렉션에 들어있는지 검사 할 수 있다.
 */
data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean {
    return p.x in upperLeft.x until lowerRight.x &&
            p.y in upperLeft.y until lowerRight.y
}

fun main(args: Array<String>) {
    val rect = Rectangle(Point(10, 20), Point(50, 50))
    println(Point(20, 30) in rect)
}
```

- 열린 범위는 끝 값을 포함하지 않는 범위이다.
- 10..20 으로 범위를 만들면 10이상 20이하인 닫힌 범위가 생기는데
- 10 until 20 으로 만들면 10이상 19이하인 열린 범위가 된다.

#### rangeTo 관례
- 범위를 만드려면 .. 구문을 사용 해야한다.
- .. 연산자는 rangeTo 함수를 간략하게 표현하는 관례이다.