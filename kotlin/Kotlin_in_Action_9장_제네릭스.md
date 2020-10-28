# Kotlin in Action

## 9장 제네릭스
- 실체화한 타입 파라미터를 사용하면, 런타임 시점에 이를 활용할 수 있다.
    - 일반 클래스나 함수는 실행 시점에 타입 정보가 사라진다.
- 선언 지점 변성을 사용할 경우 기저 타입은 동일하지만, 타입 인자가 다른 두 제네릭 타입의 상위/하위 타입 관계에 따라
- 상위/하위 타입 관계가 어떻게 되는지 지정이 가능해진다.

`기저 타입`
- List 와 List<String> 이 기반 - 파생 타입 관계라고 보기 어렵기 때문에, 제네릭 타입에서는 타입 파라미터를 제외한 부분을 기저 타입이라는 용어로 사용한다.
- 위 의 경우 List 가 기저 타입이다.

#### 제네릭 타입 파라미터
- 제네릭스를 사용하면 **타입 파라미터 (type parameter)** 를 받는 타입을 정의할 수 있다.
- 제네릭 타입의 인스턴스를 생성하려면, 타입 파라미터를 구체적인 **타입 인자 (type argument)**로 치환해야 한다.
- 코틀린 컴파일러는 일반 타입과 마찬가지로 타입 인자도 추론이 가능하다.
    - 아래 코드에서 listOf 함수에 전달된 값이 문자열 이기 때문에 컴파일러는 List<String> 임을 추론한다.
```kotlin
val authors = listOf("Dmitry", "Sveltlana")
```

> 코틀린에서는 제네릭 타입 인자를 명시 하거나 컴파일러가 추론할 수 있어야한다.
> 자바는 뒤늦게 추가되어 이전 버전 호환성을 위해 제네릭이 없는 로(raw) 타입을 허용한다.

#### 제네릭 함수와 프로퍼티
- 제네릭 함수를 선언하면 특정 타입 뿐 아닌 모든 타입을 다루는 함수를 선언할 수 있다.
- 제네릭 함수를 호출 할 때는 반드시 구체적인 타입 인자를 넘겨 주어야한다.
```kotlin
fun <T> List<T>.slice(indices: IntRange) : List<T>
```
> 함수 타입 파라미터 T는 수신 객체와 반환 타입에 사용 된다.

`제네릭 함수`
```kotlin
/**
 * 제네릭 함수를 호출 할때 구체적인 타입 파라미터를 넘겨주어야 한다.
 * 하지만 대부분의 경우 컴파일러가 타입 파라미터를 추론할 수 있기 때문에 생략이 가능하다.
 */
fun main() {
    val letters = ('a'..'z').toList()
    println(letters.slice<Char>(0..2))
    println(letters.slice(0..2))
}
```

`제네릭 확장 프로퍼티`
```kotlin
/**
 * 제네릭 함수와 마찬가지로 제네릭 확장 프로퍼티도 정의가 가능하다.
 * 아래 함수는 리스트의 마지막 원소 바로 앞에 있는 원소를 반환하는 확장 함수이다.
 */
val <T> List<T>.penultimate: T
    get() = this[size - 2]

fun main() {
    println(listOf(1, 2, 3, 4).penultimate)
}
```

> 확장 프로퍼티에만 제네릭 타입 파라미터를 사용할 수 있다.

#### 제네릭 클래스 선언
- 자바와 마찬가지로 타입 파라미터를 넣은 클래스를 선언할 수 있다.
- 타입 파라미터를 클래스 명 뒤에 선언하면, 클래스 본문 내에서 해당 타입 파라미터를 사용할 수 있다.
```kotlin
/**
 * 코틀린에서도 제네릭 클래스를 선언할 수 있다.
 */
interface List<T> {
    operator fun get(index: Int): T
}
```

- 제네릭 클래스를 확장하는 클래스를 정의하면, 기반 타입의 제네릭 파라미터에 대해 타입 인자를 지정해 주어야 한다.
```kotlin
/**
 * 제네릭 클래스를 확장하는 경우, 구체적인 타입 인자를 넘겨주거나, 확장 클래스의 타입 인자를 사용할 수 있다.
 */
class StringList: GenericList<String> {
    override fun get(index: Int): String { return "1" }
}
```

> 타입 인자를 지정할 때 자기 자신 타입을 인자로 참조할 수 있다.
> Comparable<T> 인터페이스를 구현할 때가 대표적인 예이다.

#### 타입 파라미터 제약
- **타입 파라미터 제약 (type parameter constraint)** 은 클래스 혹은 함수에 사용 가능한 타입 인자를 제한하는 기능
- 어떤 타입을 제네릭 타입의 파라미터에 대한 **상한 (upper bound)** 로 지정하면 해당 제네릭 타입 을 인스턴스화 할때 타입 인자는 
반드시 상한 타입 이거나, 하위 타입이여야 한다.
- 제약을 가하려면 타입 파라메터 명 뒤에 **콜론 (:)** 을 사용해서 상한 타입을 명시하면 된다.
- 아래 코드는 자바에서 <T extends Number> T sum(List<T> list) 와 동일하다.
```kotlin
/**
 * 타입 파라미터 제약을 가하려면 파라미터 명 뒤에 콜론 (:) 을 표시하고 그 뒤에 상한 타입을 지정하면 된다.
 */
fun <T : Number> List<T>.sum() : T = get(0)
```

> 타입 파라미터 T 에 대한 상한을 징하고 나면, T 타입의 값을 상한 타입의 값으로 취급이 가능하다.
> 상한 타입에 정의된 메소드를 T 타입 값에 대해 호출할 수 있다.

- 타입 파라미터에 둘 이상의 제약을 걸 수도 있다.
```kotlin
import java.lang.Appendable

/**
 * 타입 파라미터의 여러개의 제약 조건을 걸수도 있다.
 * 아래 예제 코드에서 T 는 CharSequence 이고, Appendable 을 구현하고 있어야 한다.
 */
fun <T> ensureTrailingPeriod(seq: T)
    where T : CharSequence, T: Appendable {
        if (!seq.endsWith('.')) {
            seq.append('.')
        }
}
```

> 둘 이상의 타입 파라미터 제약 조건이 필요하다면, where 키워드를 사용해서 지정이 가능하다.

#### 타입 파라미터를 널이 될 수 없는 타입으로 한정
- 타입 파라미터 상한을 정하지 않은 타입 파라미터는 Any? 를 상한으로 지정한 것과 동일하다.

```kotlin
/**
 * value는 널이 될 수 있기 대문에 안전한 호출을 사용해야 한다.
 */
class Process<T> {
    fun process(value: T) {
        value?.hashCode()
    }
}

/**
 * 항상 널이 될 수 없는 타입 파라미터 제약을 걸려면 상한에 Any 를 지정해야 한다.
 */
class ProcessNotNull<T : Any> {
    fun process(value: T) {
        value.hashCode()
    }
}
```

#### 실행시 제네릭스의 동작 - 소거된 타입 파라미터와 실체화된 타입 파라미터
- JVM 의 제네릭스는 **타입 소거 (type erasure)** 를 사용하여 구현된다.
    - 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않는다.
- 코틀린에서 함수를 inline 으로 선언하면, 실행 시점에 타입 인자가 지워지지 않는다.
    - 코틀린 에서는 이를 **실체화 (reify)** 라고 부른다.

#### 실행 시점의 제네릭 - 타입 검사와 캐스트
- 코틀린 제네릭 타입 인자는 자바와 동일하게 런타임 시점에 사라진다.
    - 제네릭 클래스 인스턴스가 인스턴스 생성시 사용한 타입 인자에 대한 정보를 유지하지 않는다는 의미이다.
    - 컴파일 시점에는 다른 타입으로 인식하지만, 실행 시점에는 같은 타입의 객체로 취급될 수 있다는 의미이기도 하다.
- 런타임 시점에 타입 인자 검사를 할 수 없기 때문에 런타임 시점에 List<String> 인지, List<Long> 인지 알 방법이 없다.

> 코틀린 에서는 타입 인자를 명시하지 않고 제네릭을 사용할 수 없기 때문에 **스타 프로젝션 (star projection)** 을 사용한다.

```kotlin
/**
 * 코틀린에서는 타입 인자를 명시하지 않고 제네릭을 사용할 수 없다.
 * 인자를 알 수 없는 제네릭 타입을 표현할때 스타 프로젝션 (star projection) 을 사용한다.
 * 자바의 List<?> 와 동일하다.
 */
fun main() {
    val list: List<*>? = null
}
```

- 타입 캐스팅은 기저 타입만 동일하다면, 제네릭 타입에 상관없이 캐스팅에 성공한다.
    - 컴파일 타임에 경고를 주지만, 실행하는데는 문제가 없다.
```kotlin
import java.lang.IllegalArgumentException

fun printSum(c: Collection<*>) {
    val intList = c as? List<Int>
            ?: throw IllegalArgumentException("List is expected")
    println(intList.sum())
}

fun main() {
    printSum(listOf(1, 2, 3))

    // List 기저 타입이 아니기 때문에, IllegalArgumentException 발생
    printSum(setOf(1, 2, 3))

    // List 기저 타입이기 때문에 캐스팅에는 성공하지만, sum() 함수 호출시 ClassCastException 이 발생한다.
    printSum(listOf("a", "b", "c"))
}
```
> as 혹은 as? 를 사용해서 타입 캐스팅을 시도 할 때 주의점은 **기저 타입은 같지만, 타입 인자가 달라도 캐스팅에는 성공** 한다.

#### 실체화한 타입 파라미터를 사용한 함수 선언
- inline 으로 선언된 함수의 타입 파라미터는 실행 시점에 인라인 함수의 타입 인자를 알 수 있다.
    - 인라인 함수는 컴파일 시점에 해당 함수를 호출한 식을 모두 함수 본문으로 바꾼다. (인라이닝)
    
```kotlin
/**
 * 함수를 inline 으로 선언하면, 타입 파라미터는 실행 시점에 사라지지 않는다.
 * 이를 코틀린에서는 실체화 된다고 표현한다.
 * inline 함수의 타입 파라미터에 reified 키워드를 사용하여, 실체화될 파라미터라고 알려준다.
 */
inline fun <reified T> isA(value: Any) = value is T

fun main() {
    println(isA<String>("abc"))
    println(isA<String>(123))
}
```

#### 인라인 함수에서만 실체화한 타입 인자를 사용할 수 있는 이유
- 컴파일러는 안라인 함수의 본문을 구현한 바이트 코드를 해당 함수 호출 지점에 모두 삽입한다.
- 실체화한 타입 인자를 사용해 정확한 타입 인자를 알 수 있기 때문이다.
- 이는 타입 파라미터가 아닌, 구체적인 타입을 사용하기 때문에 실행 시점에 발생하는 타입 소거의 영향을 받지 않는다.
> 자바에서는 reified 타입 파라미터를 사용하는 inline 함수를 호출할 수 없다.
> 코틀린 인라인 함수를 자바에서는 일반 함수처럼 호출하기 때문이다.

#### 실체화한 타입 파라미터로 클래스 참조 대신 사용
- java.lang.Class 타입 인자를 파라미터로 받는 API를 코틀린에서 사용하기 위해 어댑터 를 구현하는 경우 실체화한 타입 파라미터를 자주 사용한다.
- 대표적인 예로 JDK의 ServiceLoader 가 있다.

```kotlin
inline fun <reified T> loadService() = ServiceLoader.load(T::class.java)

fun main() {
    // 코틀린에서 Service::class.java 는 자바의 Service.class 와 완전히 동일한 역할을 하는 코드이다,
    val serviceImpl = ServiceLoader.load(Service::class.java)

    // 타입 파라미터 실체화를 이용한 간결한 호출함수 위의 코드와 동일한 역할을 한다.
    val serviceImpl2 = loadService<Service>()
}
```

#### 실체화한 타입 파라미터의 제약 조건
- 다음과 같은 경우 실체화한 타입 파라미터를 사용할 수 있다.
1. 타입 검사와 캐스팅
2. 코틀린 리플렉션 API
3. 코틀린 타입에 대응하는 java.lang.Class 얻기
4. 다른 함수 호출시 타입 인자로 사용

- 다음과 같은 경우는 사용할 수 없다.
1. 타입 파라미터 클래스의 인스턴스 생성
2. 타입 파라미터 클래스의 동반 객체 메소드 호출
3. 실체화한 타입 파라미터를 요구하는 함수를 호출하며 실체화 하지 않은 타입 파라미터로 받은 타입을 타입 인자로 사용
4. 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 reified 로 지정