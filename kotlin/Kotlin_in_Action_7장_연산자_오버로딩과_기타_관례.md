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

