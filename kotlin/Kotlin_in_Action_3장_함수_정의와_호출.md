# Kotlin in Action

## 3장 함수 정의와 호출

#### 코틀린에서 컬렉션 만들기
- 코틀린에서는 다양한 컬렉션을 지원한다.
- 코틀린 고유의 컬렉션이 아닌 표준 자바 컬렉션을 사용

```kotlin
/**
 * setOf 외에도 아래와 같은 방법으로 다양한 컬렉션을 생성할 수 있다.
 * map을 생성할때는 to 를 사용하는데 이는 키워드가 아닌 일반 함수이다.
 * 코틀린은 코틀린의 컬렉션을 사용하지않고 표준 자바 컬렉션을 사용한다.
 * 이는 자바와 상호작업하게 훨씬 쉽게 때문이다.
 */
val set = hashSetOf(1, 7, 53)
val list = arrayListOf(1 , 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```
```kotlin
/**
 * 컬렉션은 다양한 함수를 제공한다.
 * last() 함수는 가장 마지막 원소를 가져온다.
 * max() 는 최대 값을 가져온다.
 */
fun main(args: Array<String>) {
    val strings = listOf("first", "second", "fourteenth")
    strings.last()

    val numbers = setOf(1, 14, 2)
    numbers.max()
}
```

#### 함수를 호출하기 쉽게 만들기
- 자바의 컬렉션에 는 디폴트 toString() 구현이 되어있다.
- 이 출력형식을 커스텀 하고싶다면 Guava 혹은 Apache Commons 같은 서드파티 라이브러리를 추가하는 등 작업이 필요하다.
- 하지만 코틀린에는 이런 작업을 처리할 수 있는 함수가 표준라이브러리에 포함되어있다.

`코틀린의 편리한 함수를 사용하지 않는 방법`
```kotlin
import java.lang.StringBuilder

/**
 * 컬렉션의 toString() 을 커스터마이징 할 수 있는 함수
 * 하지만 가독성 부분에서 좋지가 않다.
 */
fun <T> joinToString(
        collection: Collection<T>,
        separator: String,
        prefix: String,
        postfix: String
) : String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

`이름이 있는 파라메터`
```kotlin
/**
 * 코틀린에서는 함수에 전달하는 인자에 이름을 명시할 수 있다.
 * 유의점은 하나라도 명시했다면 혼동을 막기 위해 그 뒤에 오는 인자는 모두 이름을 명시해야한다.
 */
fun main(args: Array<String>) {
    val list = listOf(1, 2, 3)
    // IDE 의 도움을 받을 수 있지만, 가독성 이 좋지않다.
    joinToString(list, "; ", "(", ")")

    // 코틀린에서는 위 문제를 다음과 같이 해결한다.
    joinToString(list, separator = " ", prefix = " ", postfix = ".")
}
```

`디폴트 파라메터 사용하기`

```kotlin
/**
 * 자바에서 오버로딩한 메소드가 많아진다는 문제를 코틀린에서는 디폴트 파라메터값을 통해 해결했다.
 * 함수의 디폴트 파라메터 값은 호출하는 쪽이 아닌 선언 쪽에서 지정된다는 점에 유의하라.
 */

// 자바에서 좀 더 편하게 코틀린 함수를 호출하고 싶다면 @JvmOverloads 애노테이션을 사용하라.
// 코틀린 컴파일러가 자동으로 오버로드한 메소드들을 추가해준다.
@JvmOverloads
fun <T> joinToStringWithDefaultParameter(
        collection: Collection<T>,
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
) : String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

fun main2() {
    val list = listOf(1, 2, 3)
    joinToStringWithDefaultParameter(list, "", "", "")

    joinToStringWithDefaultParameter(list, separator = ":")
}
```

#### 최상위 함수와 프로퍼티
- 자바에서는 Util 성 메소드들을 작성하기 위해 ~~Utils 라는 클래스를 생성하고 해당 부분에 모아두는 방식을 택했다.
- 코틀린에서는 이런 무의한 클래스가 필요없다.
- 그런 함수들은 그저 최상위 수준에 위치시키면 된다.
- 기억할 점은 JVM은 클래스 내에 존재하는 코드만 실행할 수 있기 때문에 컴파일러가 컴파일시 새로운 클래스를 생성해 준다는 것이다.
- 만약 최상위 함수가 포함되는 클래스명을 바꾸고 싶다면 @JvmName 애노테이션을 추가하면 된다.
- 프로퍼티도 함수와 동일하게 최상위 수준에 놓을 수 있다.
- const 키워드를 사용해 자바의 public static final ~ 과 같은 상수를 만들 수 있다.

```kotlin
const val HELLO = "HELLO"
```
```java
public static final String HELLO = "HELLO";
``` 

#### 확장 함수와 확장 프로퍼티
- 확장 함수는 어떤 클래스의 멤버 메소드처럼 호출이 가능하지만, 해당 클래스 밖에 선언된 함수이다.
- 확장 함수 생성시 추가하려는 함수 앞에 그 함수가 확장할 클래스명을 정의해야 한다.
- 클래스 명을 수신 객체 타입 (receiver type) 이라고 하며, 확장 함수가 호출되는 대상이 되는 값을 수신 객체 (receiver object) 라고 한다.
- 아래 예제엣는 String 이 수신객체 타입이고, "Hello" 가 수신 객체이다.
- 확장 함수가 캡슐화를 깨지는 않는다.
- private, protected 멤버를 사용할 수는 없다.

```kotlin
/**
 * 확장 함수를 만들때는 추가하려는 함 수앞에 그 함수가 확장할 클래스 명을 덧붙여야 한다.
 * 사용할때는 확장된 클래스의 함수를 호출하듯이 사용하면 된다.
 */
fun String.lastChar() : Char = this.get(this.length -1)

fun main(args: Array<String>) {
    println("Hello".lastChar())
}
```

#### 임포트와 확장함수
- 확장 함수를 정의해도, 해당 함수를 사용하게 위해 임포트를 해야한다.
- 하지만 이름 충돌되는 경우 이를 회피하는 방법이 필요하다.
- as 키워드를 사용하여, 임포트한 클래스나 함수를 다른이름으로 콜 할수있는데 이를 활용한 방법이다.
```kotlin
import lastChar as last

/**
 * as 키워드를 사용하여 클래스나 함수를 다른 명칭으로 사용할 수 있다.
 */
fun main(args: Array<String>) {
    "Hello".last()
}
```

#### 자바에서의 확장 함수 호출
- 내부적으로 확장함수는 수신객체를 첫 번째 인자로 받는 정적 메소드이다.
- 확장 함수를 호출해도 다른 어댑터 객체를 사용하거나 실행 시점 부가 비용이 들지 않는다.

#### 확장 함수는 오버라이드 할 수 없다
- 확장 함수는 클래스의 일부가 아니고 클래스 밖에 선언된다.
- Util 클래스를 편하게 호출하기 위한 syntax sugar 일뿐이다.
- 객체지향 메소드 오버라이드를 생각해 본다면 쉽게 이해할 수 있다.

#### 주의점
- 확장 함수와 클래스의 멤버 함수의 시그니쳐가 같다면 확장 함수가 아닌 멤버 함수가 호출된다.
- 멤버 함수의 우선순위가 더 높다.

#### 확장 프로퍼티
- 확장 프로퍼티는 기존 클래스 객체에 대한 프로퍼티 형식 구문으로 사용가능한 API 를 추가할 수 있다.
- 하지만 상태를 저장할 수 없다.

```kotlin
import java.lang.StringBuilder

/**
 * 확장 프로퍼티는 상태를 가질 수 없다.
 * 
 */
val String.lastChar: Char
    get() = get(length - 1)

var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }
``` 

#### 컬렉션 처리 - 가변 길이 인자, 중위 함수 호출, 라이브러리 지원
- 컬렉션 처리시 사용 가능한 코틀린 표준 라이브러리 함수 몇가지를 알아본다.

`코틀린 언어 특성중 3가지`
- vararg 키워드 사용시 호출시 인자 개수가 달라질 수 있는 함수를 정의 가능하다.
- 중위(infix) 함수 호출 구문을 사용하면 인자가 하나뿐인 메소드를 간편하게 호출할 수 있다.
- 구조 분해 선언을 사용하면 복합적인 값을 분해해서 여러변수에 나눠 담을 수 있다.

#### 자바 컬렉션 API 확장
- 코틀린 컬렉션은 자바와 동일한 클래스를 사용하지만 더 확장된 기능을 제공한다.
- 초반에 살펴보았던 last(), max() 함수 들은 모두 확장함수 였던 것
- 코틀린 표준 라이브러리는 수많은 확장 함수들을 포함한다.

#### 가변 인자 함수
- 리스트를 생성하는 함수 호출시 원하는 만큼 원소를 많이 전달할 수 있다.
- 함수 정의부분을 보면 vararg 를 사용했다.
```kotlin
val list = listOf(2, 3, 5, 7, 11)
fun listOf<T> (vararg values: T) List<T> {...}
```
- 배열을 가변길이 인자로 넘길때 자바는 그대로 넘기면 되지만, 코틀린은 명시적으로 풀어서 넘겨 주어야 하는데 이때 **스프레드 연산자** 가 쓰인다.

```kotlin
/**
 * 자바에서는 배열을 가변길이 인자로 넘길때 배열을 그대로 넘기면 되지만, 코틀린에서는 스프레드 연산자를 사용해야한다.
 * 단지 배열 앞에 *를 붙이기만 하면 된다.
 */
fun main(args: Array<String>) {
    val list = listOf("args", *args)
    println(list)
}
```

#### 중위 호출과 구조 분해 선언
- 맵을 생성할때 mapOf 함수를 사용한다.
- 여기서 to는 중위 호출 (infix call) 이라는 방식으로 일반 메소드를 호출한 것이다.
- 중위 호출시 수신 객체와 유일한 메소드 인자 사이에 메소드 이름을 넣는다.
- 함수를 중위 호출이 가능하게 하려면 infix 변경자를 함수 선언 앞에 추가하면 된다.

```kotlin
/**
 * 맵을 생성할 때 to 는 중위 호출을 한 예이다.
* 중위 호출을 가능하게 하려면, 함수 선언 앞에 infix 변경자를 선언해야 한다.
*/
infix fun Any.to(other: Any) = Pair(this, other)

fun main(args: Array<String>) {
    // number, name 을 1 to "one" 의 결과로 초기화 한다. 이런 방식을 구조분해선언 이라고 한다.
    val (number, name) = 1 to "one"
}
```

#### 문자열과 정규식 다루기
- 코틀린과 자바의 문자열은 같다.
- 코틀린은 다양한 확장 함수를 제공함으로써 표준 자바 문자열을 편하게 다룰수 잇게 한다.

#### 문자열 나누기
- 자바의 split 메소드로는 점(.) 을 사용해 문자열을 분리할 수 없다.
- split의 구분 문자는 실제로 정규식 이기 때문이다. 점(.)은 모든 문자를 나타내는 정규식으로 해석된다.
- 코틀린에서는 자바의 split 대신 여러 조합 파라메터를 받는 split 확장함수를 제공해서 혼동을 없앤다.
- 정규식을 파라메터로 받는 함수는 Regex 타입 값을 받는다.

```kotlin
/**
 * 코틀린에서는 자바의 split 대신 여러 조합 파라메터를 받는 split 함수를 제공함으로써 혼동을 없앤다.
 */
fun main(args: Array<String>) {
    // 정규식을 전달하는것을 명시적으로 표시
    println("12.345-6.A".split("\\.|-".toRegex()))

    // 여러 구분 문자열을 지정
    println("12.345-6.A".split(".", "-"))
}
```

#### 정규식과 3중 따옴표로 묶은 문자열
- 파일의 전체 경로명을 디렉터리, 파일명, 확장자로 구분하는 것을 구현한다.

```kotlin
/**
 * String 확장 함수를 사용하여 경로 파싱
 */
fun parsePath(path: String) {
    val directory = path.substringBeforeLast("/")
    val fullName = path.substringAfterLast("/")

    val fileName = fullName.substringBeforeLast(".")
    val extension = fullName.substringAfterLast(".")
    println("Dir: $directory, name: $fileName, ext: $extension")
}

fun main(args: Array<String>) {
    parsePath("/Users/yole/kotlin-book/chapter.adoc")
}
```

```kotlin
/**
 * 정규식을 사용한 경로 파싱
 */
fun parsePathUseRegEx(path: String) {
    val regex = """(.+)/(.+)\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path)
    if (matchResult != null) {
        val (directory, filename, extension) = matchResult.destructured
        println("Dir: $directory, name: $filename, ext: $extension")
    }
}

fun main(args: Array<String>) {
    parsePathUseRegEx("/Users/yole/kotlin-book/chapter.adoc")
}
```

#### 여러 줄 3중 따옴표 문자열
- 3중 따옴표 문자열은 문자열 이스케이프를 위해서만 사용하지 않는다.
- 줄바꿈을 표현하는 문자열에서 사용한다.

```kotlin
/**
 * 3중 따옴표 문자열을 이용한 ascii art
 */
fun main(args: Array<String>) {
    val kotlinLogo = """|  //
                       .| //
                       .|/ \""".trimMargin(".")
    println(kotlinLogo)
}
```

#### 코드 다듬기 - 로컬 함수와 확장
- 좋은 코드를 작성하는 원칙중 DRY (Don't Repeat Yourself) 원칙이 있다.
- 자바 코드작성시에는 DRY 원칙을 피하기는 쉽지 않다.
- 코틀린에서는 함수에서 추출한 함수를 원 함수 내부에 중첩시킬수 있다. 

```kotlin
import java.lang.IllegalArgumentException

class User(val id: Int, val name: String, val address: String)

/**
 * 필드 검증 로직이 중복된다.
 */
fun saveUser(user: User) {
    if (user.name.isEmpty()) {
        throw IllegalArgumentException()
    }

    if (user.address.isEmpty()) {
        throw IllegalArgumentException()
    }

    // DB SAVE
}

/**
 * 중복되는 필드 검증 로직을 로컬 함수를 이용해 제거
 * 로컬 함수는 자신이 속한 바깥 함수의 모든 파라메터와 변수에 접근할 수 있다.
 */
fun saveUserWithLocalFunction(user: User) {
    fun validate(value: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("${user.id}...")
        }
    }
    validate(user.name)
    validate(user.address)

    // DB SAVE
}

/**
 * 필드 검증 로직을 확장 함수로 User 에 등록한다.
 */
fun User.validateBeforeSave() {
    fun validate(value: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("$id...")
        }
    }
    validate(name)
    validate(address)
}

fun saveUserUseExtendFunction(user: User) {
    user.validateBeforeSave()
    // DB SAVE
}
```

> 확장 함수를 로컬 함수로 정의할 수도 있다.

#### 정리
- 코틀린은 자바 컬렉션 클래슬르 사용하되 확장 함수응 사용하여 더 많은 기능을 제공한다.
- 함수 파라메터의 디폴트 값을 정의하면 오버로딩한 함수를 정의할 필요가 줄어든다.
- 이름있는 파라메터를 사용하여 가독성을 향상시켜라.
- 최상위 함수와 프로퍼티를 직접 선언하여 이를 활용하면 코드 구조를 유연하게 만들수 있다.
- 확장 함수와 프로퍼티를 사용하여 모든 클래스의 API를 확장할 수 있다.
- 확장 함수를 호출해도 추가비용이 들지 않는다.
- 중위 호출을 통해 인자가 1개뿐인 메소드나 확장함수를 깔끔한 구문으로 호출할 수 있다.
- 정규식과 문자열 처리시 다양한 문자열 처리함수를 제공한다.
- 이스케이프가 필요한 문자열은 3중 따옴표를 사용하자.
- 로컬함수를 사용해서 코드를 깔끔하고 중복을 제거하자.