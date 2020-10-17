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
- rangeTo 함수는 범위를 반환하고, 이 연산자를 아무 클래스에나 정의할 수 있다.
- Comparable 인터페이스를 구현하고 있다면, rangeTo 를 구현하지 않아도 관례를 따를 수 있다.
- 코틀린 표준 라이브러리를 통해 생성이 가능하다.

```kotlin
import java.time.LocalDate

/**
 * RangeTo 함수를 구현하고 있다면, .. 을 통해 범위 생성 관례를 사용할 수 있다.
 * 만약 Comparable 인터페이스를 구현하고 있다면, RangeTo 함수를 구현하지 않아도 된다.
 */
fun main(args: Array<String>) {
    val now = LocalDate.now()
    val vacation = now..now.plusDays(10)
    println(vacation)
}
```

> 범위 연산자는 다른 산술 연산자보다 우선 순위가 낮기 때문에 항상 괄호 () 로 감싸는 습관을 들여라.

#### for 루프를 위한 iterator 관례
- 코틀린의 for 루프는 범위 검사와 동일하게 in 연산자를 사용한다.
- for (x in list) { ... } 은 list.iterator()를 호출해서 자바와 동일하게 이터레이터에 대해 hasNext, next 호출을 반복한다.
- 코틀린에서는 이 또한 관례라 iterator 메소드를 확장 함수로 정의할 수 있다. 
- 코틀린 표준 라이브러리 에서는 String 상위 클래스인 CharSequence 에 대한 iterator 확장 함수를 제공한다.

```kotlin
import java.time.LocalDate

/**
 * 날짜 범위에 대한 이터레이터 구현
 */
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
        object: Iterator<LocalDate> {
            var current = start

            override fun next() = current.apply { current = plusDays(1) }

            override fun hasNext() =  current <= endInclusive

        }


fun main(args: Array<String>) {
    val newYear = LocalDate.ofYearDay(2017, 1)
    val daysOff = newYear.minusDays(1)..newYear
    for (dayOff in daysOff) { println(dayOff) }
}
```

#### 구조 분해 선언과 component 함수
- 구조 분해 를 사용하면 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화할 수 있다.
- 구조 분해는 관례를 사용한다.
- 각 변수를 초기화 하기 위해 componentN 함수를 호출한다.
- N은 구조분해 선언에 있는 변수 위치에 따라 붙는 번호이다.

```kotlin
/**
 * 구조 분해를 사용하면 객체의 값을 분해해서 여러 다른 변수로 한꺼번에 초기화할 수 있다.
 */
fun main(args: Array<String>) {
    val p = Point(10, 20)
    val (x, y) = p
    println(x)
}
```

`ComponentN 함수`
```kotlin
/**
 * data 클래스의 주 생성자에 들어있는 프로퍼티는 컴파일러가 자동으로 만들어 준다.
 * componentN 함수는 구조분해 관례로 사용한다.
 */
class ComponentPoint(val x: Int, val y: Int) {
    operator fun component1() = x
    operator fun component2() = y
}

fun main(args: Array<String>) {
    val point = ComponentPoint(1, 2)
    val (x, y) = point
    println(x)
}
```

> 컬렉션이나 배열에 대해서도 componentN 함수를 제공한다. 무한히 선언할 순는 없지만 매우 유용하다.
> 코틀린 표준 라이브러리 에서는 맨 앞의 5개의 원소에 대한 componentN 을 제공한다.
> 표준 라이브러리의 Par, Triple 클래스를 사용하면 함수에서 여러 값을 더 간단하게 반환할 수 있다.

#### 구조 분해 선언과 루프
- 변수 선언이 가능한 장소라면 어디든 구조 분해 선언을 사용할 수 있다.
- 맵의 원소에 대해 이터레이션할 때 구조 분해 선언이 유용하다.

```kotlin
/**
 * 맵을 이터레이션 할 때 구조분해 문법은 매우 유용하다.
 */
fun printEntries(map: Map<String, String>) {
    for ((key, value) in map) {
        println("$key -> $value")
    }
}

fun main(args: Array<String>) {
    val map = mapOf("Oracle" to "java", "JetBrains" to "Kotlin")
    printEntries(map)
}
```

#### 프로퍼티 접근자 로직 재활용 - 위임 프로퍼티
- 코틀린이 제공하는 관례에 의존하는 특성 중 독특하면서 강력한 기능은 위임 프로퍼티 (delegated Property) 이다.
- 값을 뒷받침 하는 필드에 단순 저장하는 것 보다 복잡한 방식으로 동작하는 프로퍼티를 쉽게 구현이 가능하다.
- 프로퍼티는 위임을 사용해 값을 필드가 아닌 데이터베이스 혹은 브라우저 세션, 맵 등에 저장할 수 있다.

#### 위임 프로퍼티 소개
- 프로퍼티 위임 관례를 따르는 Delegate 클래스는 getValue, setValue 메소드를 제공 해야한다.
- 위임 프로퍼티를 사용하면 아래의 p 프로퍼티는 접근자 로직을 다른 객체에 위임 한다. by 키워드를 사용하여 위임한다.
```kotlin
class Foo {
    var p: Type by Delegate()
}

// 컴파일 된 코드
class Foo {
    private val delegate = Delegate()
    var p: Type
    set(value: Type) = delegate.setValue(..., value)
    get() = delegate.getValue(...)
}

class Delegate {
    operator fun getValue(...) { ... }
    operator fun setValue(..., value: Type) { ... }
}
```

> 코틀린 라이브러리는 프로퍼티 위임을 사용해 프로퍼티 초기화를 지연 시킬 수 있다.

#### 위임 프로퍼티 - by lazy() 를 사용한 프로퍼티 초기화 지연
- 지연 로딩은 객체의 일부분을 초기화하지 않고 필요시에만 초기화해서 사용한다.
- lazy() 함수는 위임 객체를 반환하는 표준 라이브러리 이다.
- 코틀린 관례에 맞는 getValue 메소드가 들어있는 객체를 반환한다.
- 또한 lazy 함수는 Thread-Safe 하다.
```kotlin
/**
 * by lazy() 를 사용하면 위임 프로퍼티 객체를 반환한다.
 * 지연 초기화를 사용할 수 있다.
 */
class LazyPerson(val name: String) {
    val emails by lazy { loadEmails(this) }
}

fun loadEmails(person: LazyPerson) : List<String> {
    println("${person.name} 의 메일 로딩")
    return listOf(/* .. */)
}
```  

#### 위임 프로퍼티 구현하기
- 어떤 객체를 UI 에 표시하는 경우 객체가 바뀌면 자동으로 UI 로 바뀌어야 한다.
- 자바에서는 PropertyChangeSupport, PropertyChangeEvent 클래스로 이런 처리를 자주한다.
- PropertyChangeSupport 는 리스너의 목록을 관리하고 PropertyChangeEvent 가 발생하면 모든 리스너에게 이벤트를 통지한다.


```kotlin
import java.beans.PropertyChangeListener
import java.beans.PropertyChangeSupport

open class PropertyChangeAware {
    protected var changeSupport = PropertyChangeSupport(this)

    fun addPropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.addPropertyChangeListener(listener)
    }

    fun removePropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.removePropertyChangeListener(listener)
    }
}

class NewPerson(
        val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    var age: Int = age
        set (newValue) {
            val oldValue = field
            field = newValue
            changeSupport.firePropertyChange("salary", oldValue, newValue)
        }
}

fun main(args: Array<String>) {
    val p = NewPerson("ncucu", 27, 100)
    p.addPropertyChangeListener ( PropertyChangeListener{ event ->
        println("Property ${event.propertyName} changed from ${event.oldValue} to ${event.newValue}")
    })
    p.age = 35
}
```

`도우미 클래스를 통해 프로퍼티 변경 통지 구현`
- 아래 코드는 코틀린 위임이 실제 동작하는 것과 비슷하다.
- 코틀린 표준 라이브러리에는 실제로 이와 비슷하게 동작하는 클래스가 있다.
- 하지만 자바의 PropertyChangeSupport를 사용하지는 않는다.
```kotlin
import java.beans.PropertyChangeSupport

class ObservableProperty(
    val propName: String, var propValue: Int,
    val changeSupport: PropertyChangeSupport
) {
    fun getValue() : Int = propValue
    fun setValue(newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(propName, oldValue, newValue)
    }
}

class ObservablePerson(
    val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
    val _age = ObservableProperty("age", age, changeSupport)
    var age: Int
        get() = _age.getValue()
        set(value) { _age.setValue(value) }
    val _salary = ObservableProperty("salary", salary, changeSupport)
    var salary: Int
        get() = _salary.getValue()
        set(value) { _salary.setValue(value) }
}
```

#### 프로퍼티 값을 맵에 저장
- 자신의 프로퍼티를 동적으로 정의 가능한 객체를 만들 때 위임 프로퍼티를 활용한다.
- 그런 경우를 **확장 가능한 객체 (expando object)** 라고 한다.
- by 키워드 뒤에 맵을 직접 넣으면 매우 쉽게 구현이 가능하다.

```kotlin
class Person {
    private val _attributes = hashMapOf(String, String)()
    
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }
    
    val name: String by _attributes
}
```

> 표준 라이브러리는 Map, MutableMap 에 대해 getValue, setValue 확장함수를 제공하기 때문에 위임 프로퍼티를 사용할 수 있다.

#### 정리
- 코틀린에서 정해진 함수의 이름을 오버로딩함으로서 연산자 관례를 사용할 수 있다.
- 비교 연산자 (==) 은 equals, compareTo 메소드로 변환된다.
- get, set, contains 함수를 정의하면 해당 인스턴스에 [], in 연산을 사용할 수 있다.
- rangeTo, iterator 함수를 정의하면 범위를 만들거나, 컬렉션과 배열의 원소를 이터레이션할 수 있다.
- 구조분해 선언을 통해 객체의 상태를 분해해서 한번에 여러 변수 초기화가 가능하다. 데이터 클래스에서는 이미 구현이 되어 있지만
- 커스텀 클래스에서는 componentN 함수를 정의해야 한다.
- 위임 프로퍼티를 통해 프로퍼티 값을 저장하거나 초기화하거나 읽거나 변경하는 로직을 재활용할 수 있다.
- lazy() 함수를 통해 지연 초기화 프로퍼티를 쉽게 구현할 수 있다.
- 맵을 위임 객체로 사용하는 위임 프로퍼티를 통해 다양한 속성을 제공하는 객체를 유연하게 다룰수 있다. 


