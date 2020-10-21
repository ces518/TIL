# Kotlin in Action

## 8장 고차함수: 파라미터와 반환 값으로 람다 사용


#### 고차 함수 정의
- 고차 함수는 다른 함수를 인자로 받거나 함수를 반환하는 함수이다.
- 코틀린에서는 람다나 함수 참조를 사용해 함수를 값으로 표현할 수 있다.

#### 함수 타입
- 코틀린에서 함수 타입을 정의하려면, 함수 파라미터의 타입을 괄호 안에 넣고, 그 뒤에 화살표를 추가한 뒤 함수 반환타입을 지정하면 된다.
- (Int, String) -> Unit
- 반환 타입이 널이 거나, 함수 타입 전체가 널일 수 있는 변수를 선언할 때는 주의해야 한다.
- (Int, Int) -> Int? : 반환 타입이 널
- ((Int, Int) -> Int)? : 함수 타입 전체가 널

#### 파라미터 이름과 함수 타입
- 함수 타입에서 파라미터 명을 지정할 수 있다.
- 이는 강제적인 것은 아니며, 호출하는 쪽에서 원하는 파라미터 명으로 변경하여 사용이 가능하다.
```kotlin
/**
 * 함수의 파라미터 명을 지정할 수 있으며, 클라이언트에서 원하는 파라미터 명으로 변경해서 사용이 가능하다.
 */
fun performRequest(
        url: String,
        callback: (code: Int, content: String) -> Unit
) {
    /* .. */
}

fun main() {
    val url = "http://www.naver.com"
    performRequest(url) { code, content -> /*..*/ } // api 에서 정의한 파라미터 명을 사용
    performRequest(url) { code, page -> /*..*/ } // 원하는 이름으로 변경해서 사용 가능
}
```

#### 인자로 받은 함수 호출
- 간단한 고차함수 구현하기
- operation 함수를 인자로 받아 해당 함수를 실행하는 심플한 예제
```kotlin
/**
 * 심플한 고차함수 정의
 */
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    println("The Result is $result")
}

fun main() {
    twoAndThree { a, b -> a + b}
    twoAndThree { a, b -> a * b}
}
```

`filter 함수 선언`
```kotlin
fun String.filter(predicate: (Char) -> Boolean) : String
```
- 표준 라이브러리인 filter 함수는 고차 함수를 사용하는 예중 하나이다.
- filter 함수는 술어를 파라미터로 받는다.
- predicate 파라미터는 문자를 인자로 받아 Boolean 값을 반환한다.
- true/false 여부에 따라 결과 목록에서 제외할 수 있다.

#### 자바에서 코틀린 함수 타입 사용
- 코틀린 함수 타입은 컴파일 되면 일반 인터페이스로 변경된다.
- 함수 타입의 변수는 **FunctionN 인터페이스** 구현체의 인스턴스 저장한다.를
- 코틀린 표준 라이브러리는 함수 인자의 개수에 따라 Function0()<R>, Function1()<P1, R> 등의 인터페이스를 제공한다.
- 해당 인터페이스는 invoke 메소드 하나만 정의되어 있다.

> 함수 타입을 사용하는 코틀린 함수를 자바에서도 쉽게 호출할 수 있다.
> 자바8 람다를 넘기면 함수 타입이 값으로 변환된다.

- 코틀린 표준라이브러리의 확장함수를 자바에서도 호출이 가능하지만, 깔끔한 호출은 되지 않는다.
- 또한 Unit 타입의 반환을 명시적으로 해주어야 하며, 자바의 void 와 대응하지 않는다.
```java
List<String> strings = new ArrayList<>();
CollectionsKt.forEach(strings, s -> {
    /* .. */
    return Unit.INSTANCE;
})
```

#### 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터
- 함수타입을 파라미터로 선언할 경우에도 다른 파라미터 처럼 디폴트 값 지정이 가능하다.

`Function Paramter With DefaultValue`
```kotlin
/**
 * 3장에서 살펴보았던 예제의 일부를 채용
 * 함수를 파라미터로 선언할 경우 다른 파라미터와 동일하게 디폴트값 설정이 가능하다.
 */
fun <T> Collection<T>.joinToString(
        transform: (T) -> String = { it.toString() }
) : String {
    /* .. */
    val result = StringBuilder()
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(element)
        result.append(transform(element))
    }
    return result.toString()
}
```

- 만약 함수타입이 nullable 하다면, 다른 변수와 마찬가지로 안전한 호출이 가능하다.

`Function Parameter With Nullable`
```kotlin
/**
 * 함수 파라미터가 nullable 하다면, 다른 변수들과 동일하게 안전한 호출을 사용할 수 있다.
 * 아래 예제는 안전한 호출과 엘비스 연산자를 활용한 예이다.
 */
fun <T> Collection<T>.joinToStringWithNullableFunction(
        transform: ((T) -> String)? = null
) : String {
    /* .. */
    val result = StringBuilder()
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(element)
        result.append(
                transform?.invoke(element) ?: element.toString()
        )
    }
    return result.toString()
}
```

#### 함수에서 함수를 반환
- 함수내에서 함수를 반환하는 경우는 적지만, 매우 유용하다.
- 특정 조건에 따라 달라질수 있는 로직을 함수를 반환하는 함수를 정의하고
- 해당 함수를 호출하는 구조를 가짐으로 깔끔한 코드 스타일로 가져갈 수 있다.

```kotlin
/**
 * 함수에서 함수를 반환하는 경우는 많지 않지만
 * 매우 유용하다.
 *
 * 특정 조건에 따라 로직이 분기되는 경우 함수를 반환하는 함수를 활용해서 깔끔한 코드 스타일로 가져갈 수 있다.
 *
 */
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

fun getShippingCostCaculator(
        delivery: Delivery
) : (Order) -> Double {

    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }
    return { order -> 61.2 * order.itemCount }
}

fun main() {
    val calculator = getShippingCostCaculator(Delivery.EXPEDITED)
    println("Shipping costs ${calculator(Order(3))}")
}
```

#### 람다를 활용한 중복 제거
- 함수 타입과 람다 식은 재활용하기 좋은 코드를 만들기 좋다.
- 다음은 웹사이트 방문 기록을 분석하는 예제 코드이다. 

```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/singup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)

/**
 * 사이트 방문 데이터 분석 V1
 * 컬렉션 api 를 이용한 보편적인 방법
 */
fun processV1() {
    val average = log
            .filter { it.os == OS.WINDOWS }
            .map(SiteVisit::duration)
            .average()
    println(average)
}

/**
 * 사이트 방문 데이터 분석 V2
 * 확장 함수를 이용해 중복을 제거
 */
fun processV2() {
    fun List<SiteVisit>.averageDurationFor(os: OS) =
            filter { it.os == os }.map(SiteVisit::duration).average()

    println(log.averageDurationFor(OS.WINDOWS))
    println(log.averageDurationFor(OS.MAC))
}

/**
 * 사이트 방문 데이터 분석 V3
 * 고차 함수를 이용한 중복 제거
 * 중복 제거뿐이 아닌 필터 조건이 변경되어도 대응이 가능하다
 */
fun processV3() {
    fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
            filter(predicate).map(SiteVisit::duration).average()

    println(log.averageDurationFor { it.os == OS.WINDOWS })
    println(log.averageDurationFor { it.os == OS.MAC })
    println(log.averageDurationFor { it.os in setOf(OS.ANDROID, OS.IOS) })
    println(log.averageDurationFor { it.os == OS.IOS && it.path == "/signup" })
}
```

> 코드 중복을 줄일 때 함수 타입은 매우 유용하다.
> 코드의 일부분을 복사해 붙여넣고 싶은 경우가 있다면 해당 코드를 람다로 만들어 중복을 제거할 수 있다.

#### 인라인 함수 - 람다의 부가 비용 없애기
- 코틀린은 보통 람다를 무명 클래스로 컴파일 한다.
- 하지만 람다 식을 상요할 때 마다 새로운 클래스가 생성되는 것은 아니다.
- 람다가 변수를 포획하는 경우에만 생성 시점마다 새로운 무명 클래스 객체가 생성된다.
- 이런 문제를 해결하기 위해 코틀린에서는 inline 함수 기능을 제공한다.

#### 인라이닝이 동작하는 방식
- 함수를 inline 으로 선언하면, 해당 함수의 본문이 인라인 된다.
- 이는 함수를 호출하는 코드 대신 함수 본문을 변역한 바이트 코드로 컴파일 한다.

`Synchronized 예제`
```kotlin
/**
 * inline 함수를 사용하면 함수 호출시 함수 본문이 인라인 된다.
 */
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    } finally {
        lock.unlock()
    }
}

fun main() {
    val l = Lock()
    synchronized(l) {
        /* .. */
    }
}
```

> 인라인 된 함수 뿐이 아닌, 전달된 람다의 본문도 함께 인라이닝 된다.

`전달된 람다가 inline 되지 않는 경우`
```kotlin
class LockOwner(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) {
        synchronized(lock, body)
    }
}
```

> 위와 같은 경우 람다의 코드를 알 수 없기 때문에 전달된 람다는 인라인 되지 않는다.

#### 인라인 함수의 한계
- 인라이닝 하는 방식으로 람다를 사용하는 모든 함수를 인라이닝 할 수는 없다.
- 파라미터로 받은 람다를 다른 변수에 저장하고 나중에 해당 변수를 사용한다면 람다를 인라이닝 할 수 없다.

`둘 이상의 람다를 인자로 받는 함수에서 일부만 인라이닝 하는 경우`
```kotlin
/**
 * 인라인 함수의 인자로 받는 람다 중 인라이닝 하고 싶지 않은 경우 noinline 키워드를 사용해 인라이닝하지 않을 수 있다.
 */
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {

}
```

#### 컬렉션 연산 인라이닝
- 코틀린 표준 라이브러리의 컬렉션 함수는 대부분 람다를 인자로 받는다.
- 인라인 함수이기 때문에 해당 함수의 바이트 코드는 인자로 전달받은 **람다 본문과 함께 인라이닝** 된다.
- 만약 컬렉션 함수를 다음과 같이 연쇄해서 사용하면 어떻게 될까 ?
```kotlin
val people = listOf(Person("ncucu", 10), Person("ncucu2", 11))
people.filter { it. age > 10 }.map(Person::name)
```
- filter 와 map은 인라인 함수이기 때문에 두 함수의 본문은 인라이닝 된다.
- 위 코드는 리스트를 걸러낸 결과를 저장하는 중간 리스트를 만든다.
- 만약 asSequence를 통해 중간 리스트를 사용하지 않고 연쇄적으로 사용하게 되면, 중간 리스트로 인한 부가비용은 줄어들게 된다.
- 하지만 시퀀스는 람다를 저장해야 하기 때문에 람다를 인라이닝 하지 않는다.
- 중간 리스트를 없앰으로써 **성능을 향상시킬 목적으로 asSequence를 사용하는것은 크기가 작은 컬렉션에서는 오히려 성능 저하**가 될 수 있다.

#### 함수를 인라인으로 선언해야 하는 경우
- inline 함수는 람다를 인자로 받는 함수만 성능이 좋아질 가능성이 높다.
- 일반적인 함수 호출의 경우 JVM은 이미 강력한 인라이닝을 지원한다.
- 또한 inline 함수는 매우 작아야 한다.
- 인라인 함수의 크기가 크다면 그만큼 바이트코드도 커질 것이다.

#### 자원 관리를 위해 인라인된 람다 사용하기
- 람다로 중복을 제거하는 패턴 중 하나는 어떤 작업 전 자원을 획득하고, 작업을 마친 뒤 자원을 회수하는 반복적인 부분에 대해 적용하는 것이다.
- 여기서 말하는 자원 (resource) 은 파일, 락, 트랜잭션 등을 의미한다.
- try-with-resource 같은 패턴은 코틀린에서 언어적으로 지원하지 않고, 함수로서 제공한다.
```kotlin
/**
 * 코틀린 에서는 언어적인 차원에서 자바의 try-with-resource 구문을 지원하지 않는대신
 * use 라는 함수를 이용해서 동일한 기능을 지원한다.
 */
fun readFirstLineFromFile(path: String): String =
    BufferedReader(FileReader(path)).use { it.readLine() }
```
> use 함수는 closeable 자원에 대한 확장 함수이며 람다를 인자로 받는다.

#### 람다 내의 return 문 - 람다를 둘러싼 함수로부터 반환
- 인라인됨 람다 내에서 return은 넌로컬 return 이다.
- 이는 함수가 인라이닝 될 때 어떻게 동작하는지 생각해본다면 당연한 일이다.
```kotlin
data class Person(val name: String, val age: Int)

/**
 * 인라인된 람다 내에서의 return 은 넌로컬 return이다.
 * 이는 람다가 아닌 람다를 감싸고 있는 함수르 return 시킨다.
 * 인라인 된 코드가 어떻게 동작하는지 생각해본다면 당연한 것이다.
 */
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Find Alice")
            return
        }
    }
    println("Not Found")
}
```

#### 람다로부터 반환 - 레이블을 사용한 return
- 람다식 내에서도 로컬 return 을 사용할 수 있다.
- 람다 내에서의 로컬 return은 for-루프의 break와 비슷한 역할을 수행한다.
- 로컬과 넌로컬 return을 구분하기 위해 레이블 (label) 을 사용해야 한다.
- return으로 실행을 끝내고 싶은 람다식 앞에 레이블을 붙이고, return 키워드 뒤에 해당 레이블을 추가하면 된다.
```kotlin
/**
 * 람다식에 레이블을 붙이려면 람다를 여는 블록 {} 앞에다가 넣으면 레이블을 붙일 수 있다.
 * return문 뒤에 레이블을 사용하면 로컬 return이 된다.
 */
fun main() {
    val strings = listOf("A", "B", "C")
    strings.forEach label@{
        if (it == "C") return@label
        println("hello")
    }
    println("ddd")
}
```

> 레이블 뿐 아니라 인라인 함수의 이름을 return 뒤에 레이블로 사용해도 동일하게 동작한다. 또한 람다식에는 레이블이 2개이상 붙을 수 없다.

#### 무명함수 - 기본적으로 로컬 return
- 넌 로컬 반환문은 장황하며, 람다 안의 여러 위치에 return 식이 들어갈 경우 사용하기 불편한다.
- 무명함수를 사용하면 넌로컬 반환문을 많이 사용해야 할 경우 코드블록을 쉽게 작성할 수 있다.
```kotlin
/**
 * 로컬 return을 많이 사용해야 한다면, 코드가 장황해질 수 있다.
 * 이런 경우 무명함수를 사용하는것을 추천한다.
 * 무명함수는 기본적으로 로컬 return이다.
 */
fun main() {
    val strings = listOf("A", "B", "C")
    strings.forEach(fun (str) {
        if (str == "C") return
        println(str)
    })
}
```

> return 의 규칙은 fun 키워드를 통해 정의된 가장 안쪽 함수를 반환한다는 점이라는 것을 기억하라.
> 무명함수 또한 람다 식의 구현 방법, 람다 식을 인라인 함수에 넘길때 인라이닝 되는 규칙 등을 모두 적용할 수 있다.


#### 정리
- 함수 타입을 사용해 함수에 대한 참조를 담는 변수, 파라미터, 반환 값을 만들 수 있다.
- 고차 함수는 다른 함수를 인자로 받거나, 함수를 반환한다.
- 인라인 함수를 컴파일 하면 해당 함수의 본문과 함수에 전달된 람다의 본문까지 인라이닝 된다.
- 고차 함수를 사용하면 각 코드의 재사용성을 높혀준다.
- 인라인 함수 내에서 넌 로컬 return 을 사용할 수 있고, 로컬 return 기능또한 제공한다.
- 람다 대식 무명함수를 사용할 수 있으며, 로컬 return 이 많다면, 무명함수를 사용하라.