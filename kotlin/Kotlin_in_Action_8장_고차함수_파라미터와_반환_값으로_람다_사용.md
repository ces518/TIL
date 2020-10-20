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

#### 인라인 함수 - 람다 부가비용 없애기
- 코틀린은 보통 람다를 무명 클래스로 컴파일 한다.
- 하지만 람다식 사용시 마다 매번 새로운 클래스가 만들어지지는 않는다.
- 람다가 변수를 포획하면 실행 시점마다 새로운 클래스가 생성 되기 때문에 부가 비용이 든다.
- 이를 해결하기 위해 코틀린에서는 inline 방식을 제공한다.

#### 인라이닝이 작동하는 방식
- 함수를 inline 으로 선언한다면 해당 함수의 본문이 인라인 된다.
- 이는 함수를 호출하는 코드를 함수 본문을 번역한 바이트코드로 바꿔치기 해버린다.

`synchronized inline 함수 예제`
```kotlin
import sun.misc.Lock

/**
 * inline 함수를 정의하면 해당 함수를 호출하는 코드는 실행 시점에 함수 본문으로 바꿔치기 해버린다.
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
    /*
    위 코드는 다음과 같이 바이트코드로 변환된다.

    l.lock()
    try {
        
    } finally {
        l.unlock()
    }
    */
}
```
> 인라인 함수의 본문만이 아닌, 해당 함수에 전달된 람다의 본문도 함께 인라이닝 된다.

#### 인라인 함수의 한계