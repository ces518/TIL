# Kotlin in Action

## 6장 코틀린 타입 시스템

#### 널 가능성
- 널 가능성 (Nullability) 은 NPE 를 피하기 위해 코틀린 타입 시스템의 특성이다.
- 코틀린을 비롯한 최신 언어에서는 null을 컴파일시 미리 감지해 실행 시점에 발생할 예외 가능성을 줄이는 방법을 택했다.

#### 널이 될 수 있는 타입
- 가장 중요한 차이는 코틀린 타입 시스템은 널이 될 수 있는 타입을 명시적으로 지원한다.
- 널이 될수 있는 변수에 대해 메소드를 호출을 하지 못하게 함으로써 많은 오류를 방지한다.

```kotlin
/**
 * null 이 올수 있는 타입을 넣을 경우 컴파일 에러를 발생시켜 NPE를 방지한다.
 */
fun strLen(s: String) = s.length
fun strLenNullable(s: String?) = s.length

fun main(args: Array<String>) {
    // null 이 올 수 있는 타입을 넣을 경우 컴파일 에러가 발생한다.
//    strLen(null)
    
    strLenNullable(null)
}

```

- 함수가 널과 문자열을 인자로 받을 수 있게 하려면 타입 이름 뒤 물음표를 명시해야 한다.

> 어떤 타입이든 타입 뒤에 물음표를 붙이면 null 참조를 저장할수 있다.
> 물음표가 없다면 null 참조를 저장할 수 없다.

- 널을 체크하기 위한 도구가 if 뿐이라면 코드가 매우 난잡해질 것이다.
- 코틀린은 널 체크를 위한 여러 도구를 제공한다.

#### 타입의 의미
- 타입이란 무엇이고 왜 변수에 타입을 지정해야 할까 ?
- 타입은 분류 (classification), 타입은 어떤 값들이 가능한지와 그 타입에 대해 연산을 수행할 수 있는 종류를 결정한다.
- 코틀린의 널이 될 수 있는타입은 여러가지 문제에 대한 종합적인 해법을 제공
- 각 타입 값에 대해 어떤 연산이 가능할지 명확히 이해할 수 있고, 실행 시점에 예외가 발생할 수 있는 연산 판단이 가능하다.

> 실행 시점에 nullable 과 nullable 하지않은 타입의 객체는 같다.
> 모든 검사는 컴파일 시점에 수행되기 때문에 별도 실행 시점 부가비용이 들지 않는다.

#### 안전한 호출 연산자: ?.
- 코틀린이 제공하는 도구중 하나인 안전한 호출연산자인 ?. 이다.
- ?. 는 null 검사와 메소드 호출을 한번의 연산으로 수행한다.
- s?.toUpperCase() 는 if (s != null) s.toUpperCase() else null 과 같다.
- 이 연산자는 연쇄적으로 호출이 가능하다.
```kotlin
/**
 * Null Safe 연산자를 사용하면 좀 더 간결하게 표현할 수 있다.
 */
fun printAllCaps(s: String?) {
    val allCaps: String? = s?.toUpperCase()
    println(allCaps)
}

fun main(args: Array<String>) {
    printAllCaps("abc")
    printAllCaps(null)
}
```

#### 엘비스 연산자: ?:
- 코틀린은 null 대신 사용할 디폴트 값을 지정할 때 편한 연산자를 제공하는데 이를 **엘비스 연산자** 라고 한다.
- 90도 돌리면 엘비스 프레슬리의 특유 헤어스타일, 눈과 닮아서 엘비스 연산자라고 부른다.

```kotlin
/**
 * 엘비스 연산자는 디폴트 값을 지정할때 유용하게 사용한다.
 * 좌항의 값이 널이라면 디폴트값을 결과로하고, 좌항의 값이 널이아니라면 좌항의 값이 결과가 된다.
 */
fun foo(s: String?) {
    val t: String = s ?: "" // s 가 널이라면 결과는 빈문자열이다.
}

fun strLenSafe(s: String?): Int = s?.length ?: 0

fun main(args: Array<String>) {
    strLenSafe(null)
    strLenSafe("abc")
}
```

#### 안전한 캐스트: as?
- as 사용시 마다 is 를 통해 매번 검사하는것은 코틀린 스럽지 않은 방법이다.
- as? 연산자는 어떤 값을 지정한 타입으로 캐스트하며, 대상 타입으로 변환할수 없다면 null 을 반환한다.
- 안전한 캐스트를 사용할때 보통 수행 한뒤 엘비스 연산자를 사용한다.

```kotlin
/**
 * 안전한 캐스트 연산자를 이용한 equals 구현 패턴
 * 보통 안전한 캐스트를 사용할때 뒤에 엘비스 연산자를 사용하는 패턴을 많이 사용한다.
 */
class SafePerson(val firstName: String, val lastName: String) {
    override fun equals(other: Any?): Boolean {
        val otherPerson = other as? SafePerson ?: return false
        return otherPerson.firstName == firstName &&
                other.lastName == lastName
    }
}
```

> 안전한 호출, 안전한 캐스트, 엘비스 연산자는 유용하기 때문에 코틀린 코드에 자주 나타난다.

#### 널 아님 단언: !!
- 널 아님 단언 (not-null assertion) 은 코틀린에서 널이 될 수 있는 타입의 값을 다룰때 가장 단순하면서 무딘 도구이다.
- 느낌표를 이중으로 사용하면 어떤 값이든 널이 될 수 없는 타입으로 바꿀 수 있다.
```kotlin
/**
 * NotNullAssertion 은 nullable한 타입의 값을 nullable 하지 않게 강제로 바꾼다.
 * 코틀린 개발자들은 컴파일러가 검증할 수 없는 단언을 사용하기 보다 더 나은 방법을 찾아보라는 의드로
 * !! 라는 기호로 표현하였다.
 */
fun ignoreNulls(s: String?) {
    val sNotNull: String = s!!
    println(sNotNull.length)
}
```

> 널 아님 단언문이 더 나은 해법인 경우도 있다.
> 어떤 값이 널인지 파악하기 위해 여러 !! 단언문을 한줄에 쓰는것을 피해야 한다.

#### let 함수
- let 함수를 사용하면 널이 될수 있는 식을 더 쉽게 다를 수 있다.
- 안전한 호출 연산자와 함께 사용하면 원하는식을 평가해서 결과가 널인지 검사한 뒤 
- 그 결과를 변수에 넣는 작업을 간단한 식으로 처리할 수 있다.
- let 함수를 통해 인자를 전달할 수도 있다.
- let 함수는 자신의 수신객체를 인자로 전달받은 람다에게 넘긴다.
```kotlin
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

/**
 * let과 안전한 연사자를 사용해 null 객체를 다룰때 좀 더 간결해지는 예제이다.
 */
fun main(args: Array<String>) {
    var email: String? = "ncucu.me@kakaocommerce.com"

    // email 이 null 이 아닐경우 let 의 람다로 전달되어 sendEmailTo 함수가 호출된다.
    email?.let { sendEmailTo(it) }
}
```

> 여러 값이 널인지 검사해야 한다면 let 을 중첩해서 할 수 있지만, 그런 경우 복잡해져 알아보기 힘들다.
> if 를 사용해 검사하는 편이 낫다.

#### 나중에 초기화할 프로퍼티
- 객체 인스턴스를 생성한 뒤 나중에 초기화하는 프레임워크가 많다.
- 예를 들어 JUnit 에서는 @Before 애노테이션이 적용된 메소드 내에서 초기화 로직을 수행한다.
- 코틀린에서는 클래스 안의 널이 될수없는 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메소드 안에서 초기화 할 수 없다.
- 하지만 이런 경우 코드가 난잡해져 보기 힘들어진다.
- 이를 위해 lateinit 변경자를 지원한다.

```kotlin
class MyService {
    fun performAction(): String = "foo"
}

/**
 * lateinit 변경자를 사용해 늦은 초기화를 할 수 있다.
 */
class MyTest {
    private lateinit var myService: MyService

    @Before
    fun setUp() {
        myService = MyService()
    }

    @Test
    fun testAction() {
        myService.performAction()
    }
}
```

- lateinit 프로퍼티를 초기화 하기전에 접근하면 has not been initialized 예외가 발생한다.

> lateinit 프로퍼티는 항상 var 여야만 한다.

#### 널이 될 수 있는 타입 확장
- 널이 될수 있는 타입에 대한 확장 함수를 정의하면 강력한 도구로 활용할 수 있다.
- 직접 변수에 대해 메소드를 호출해도 확장함수인 메소드가 알아서 널을 처리해준다.
- 이런 처리는 확장 함수에서만 가능하다.
- 일반 멤버 호출은 인스턴스를 통해 dispatch 되므로 널 여부를 검사하지 않는다.

`디스패치 란 ?`
- 객체지향 언어에서 동적 타입에 따라 적절한 메소드를 호출해주는 것을 동적 디스패치 라고 한다.
- 반면 컴파일 시점에 결정하여 코드를 생성하는 방식을 지겁 디스패치 라고 한다.
- 동적 디스패치는 객체별로 자신의 메소드에 대한 테이블을 저장하는 방법을 많이 사용한다.

```kotlin
/**
 * 널이 될수 있는 수신 객체에 대한 확장 함수 호출
 * 안전한 호출(?.) 없이도 널이 될 수 있는 수신 객체 타입에 대해 선언된 확장함수 호출이 가능하다.
 * null 이 들어오는 겨우를 적절히 처리한다.
 */

fun String?.isNullOrBlank(): Boolean =
        this == null || this.isBlank()

fun verifyUserInput(input: String?) {
    if (input.isNullOrBlank()) {
        println("Please fill in the required fields")
    }
}

fun main(args: Array<String>) {
    verifyUserInput("")
    verifyUserInput(null)
}
```

> 자바 메소드 내에서 this는 항상 널이 아니지만, 코틀린 에서는 **널이 될수 있는 타입의 확장 함수 안에서 this 는 널이 될 수 있다.**

#### 타입 파라미터의 널 가능성
- 코틀린에서 함수나 클래스의 모든 타입 파라미터는 널이 될 수 있다.
- 타입 파라미터 T를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라도 널이 될수 있는 타입이다.

```kotlin
/**
 * 코틀린에서 타입 파라미터는 널이 될 수 있는 타입이다.
 * 타입 뒤에 ? 가 없더라도 널이 될 수 있는 타입이라는 점을 기하라.
 */
fun <T> printHashCode(t: T) {
    println(t?.hashCode())
}

fun main(args: Array<String>) {
    printHashCode(null)
}
``` 

- 타입 파라미터가 널이 아님을 확실히 하려면 **널이 될 수 없는 타입 상한 (upper bound) 를 지정**해야 한다.
```kotlin
/**
 * 타입 파라미터가 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상한을 지정해야한다.
 */
fun <T: Any> printHashCodeNotNull(t: T) {
    println(t.hashCode())
}
```

#### 널 가능성과 자바
- 코틀린은 자바 상호운용성을 강조하는 언어이다.
- 자바 타입 시스템은 널 가능성을 지원하지 않는다.
- 자바와 코틀린을 조합했을때 매번 null 을 검사 해야하나 ?
    - 자바 코드에도 애노테이션으로 표시된 널 가능성 정보가 있다. 그런 정보가 있는경우 코틀린에서도 이를 활용한다.
- 코틀린은 여러 널 가능성 애노테이션을 알아보며, 널 가능성 애노테이션이 없는 경우 자바의 타입은 코틀린의 플랫폼 타입 이된다.

#### 플랫폼 타입
- 플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입을 말한다.
- 널이 될수 있는 타입, 널이 될수 없는 타입 둘중 아무렇게나 처리해도 된다.
- 자바와 마찬가지로 플랫폼 타입에 대해 수행하는 모든 연산의 책임은 개발자에게 있다는 의미이다.
- 코틀린은 플랫폼 타입에 대해 수행하는 연산은 경고를 표시하지 않는다.
- 코틀린 컴파일러는 public 가시성인 코틀린 함수의 널이 아닌 타입 파라미터와 수신 객체에 대해 널 검사를 추가해준다.
- 파라미터 값 검사는 호출 시점에 이루어 진다.

> 자바 API 를 다룰 때는 조심해야 한다. 대부분 널 관련 애노테이션을 쓰지 않기 때문에 플랫폼 타입이다.

#### 코틀린이 플랫폼 타입을 도입한 이유
- 모든 자바 타입을 널이 될 수 있는 타입으로 다루면 더안전하지만
- 널이 될수 없는 값에 대해서도 불필요한 널검사가 들어가기 때문이다.

> 코틀린에서 플랫폼 타입을 선언할 수는 없다.

#### 상속
- 코틀린에서 자바 메소드를 오버라이드 할 때 그 메소드의 파라미터와 반환 타입을
- 널이 될 수 있는 타입인지, 널이 될수 없는 타입인지 선언해주어야 한다.
- 자바 클래스나 인터페이스를 코틀린에서 구현할 경우 널 가능성을 제대로 처리하는것이 중요하다.
- 코틀린 컴파일러는 구현 메소드를 널이 될 수 없는 타입으로 선언한 모든 파라미터에 대해 널이 아님을 검사하는 단언문을 생성해준다.

#### 코틀린 원시 타입
- 코틀린은 원시 타입과 래퍼 타입을 구분하지 않는다.

#### 원시 타입 - Int, Boolean 등..
- 자바는 원시 타입과 참조 타입을 구분한다.
- 원시 타입의 변수에는 그 값이 직접 들어가지만, 참조 타입은 변수의 주소값이 들어간다.
- 코틀린은 원시 타입과 래퍼 타입을 구분하지 않고 항상 같은 타입을 사용한다.
- 항상 객체로 표현한다면 매우 비효율 적이겠지만 코틀린은 그렇지 않다.
- 실행 시점에 숫자 타입은 가능한 가장 효율적인 방식으로 표현된다.
    - 대부분의 경우 코틀린 Int 타입은 자바 int 타입으로 컴파일 된다.
- Int 와 같은 코틀린 타입에는 널 참조가 들어갈 수 없기 대문에 그에 상응하는 자바 원시 타입으로 컴파일 된다.
- 자바 원시타입을 코틀린에서 사용할 때도 플랫폼 타입이 아닌 널이 될수 없는 타입으로 취급 된다.

#### 널이 될 수 있는 원시타입: Int?, Boolean? 등
- 코틀린의 널이 될 수 있는 원시타입을 사용하면 자바의 래퍼타입으로 컴파일 된다.
- 제네릭 클래스의 경우 래퍼타입을 사용한다.
- 이는 JVM 에서 제네릭을 구현하는 방법 때문이다.
- JVM 은 타입 인자로 원시 타입을 허용하지 않기 때문에 코틀린 자바 모두 제네릭 클래스는 항상 박스 타입을 사용 해야 한다.
```kotlin
/**
 * 널 가능성이 있는 타입의 두 값을 직접 비교할 수는 없다.
 * 널 검사를 마친 뒤 일반 값처럼 다루게 허용된다.
 */
data class HelloPerson(val name: String,
                       val age: Int? = null) {
    fun isOlderThan(other: HelloPerson): Boolean? {
        if (age == null || other.age == null) {
            return null
        }
        return age > other.age
    }
}
```

#### 숫자 변환
- 코틀린과 자바의 가장 큰 차이점 중 하나는 숫자를 변환하는 방식이다.
- 코틀린은 다른 타입의 숫자로 자동 변환하지 않는다.
- 결과 타입이 허용하는 범위가 원래 타입 범위보다 넓은 경우에도 자동 변환을 하지 않는다.
- 직접 변환 메소드를 호출해야 한다. 
```kotlin
/**
 * 코틀린은 숫자 자동변환을 지원하지 않는다.
 * 직접 변환함수를 호출 해주어야 한다.
 */
fun main(args: Array<String>) {
    // 컴파일 에러, 자동 변환 하지 않음
    val i = 1
    val l: Long = i

    // 직접 변환 함수를 호출 해주어야 한다.
    val i2 = 1
    val l2: Long = i2.toLong()
}
```

> 코틀린은 Boolean 을 제외한 모든 원시 타입에 대한 변환 함수를 제공한다.

#### Any, Any?: 최상위 타입
- 자바에서 Object 가 최상위 타입이듯, 코틀린에서는 Any 타입이 모든 널이 될 수 없는 타입의 조상이다.
- 자바에서는 원시타입 대신 래퍼클래스만 Object 를 조상으로 가지지만, 코틀린은 그렇지 않다.
- 코틀린에서 원시타입 값을 Any 타입 변수에 대입하면 자동으로 값을 객체로 감싼다. 

```kotlin
/**
 * 자바에서 Object 가 최상위 클래스인것 처럼 코틀린에서는 Any가 최상위 클래스이다.
 * 차이점 이라면 원시타입의 조상도 Any 라는 것이다.
 */
fun main(args: Array<String>) {
    val answer: Any = 42
}
```
> 코틀린 Any 타입을 사용하면 자바 Object 타입으로 컴파일 된다.

#### Unit 타입 - 코틀린의 Void
- Unit 타입은 자바의 void 와 같은 기능을 한다.
- 반환 타입 선언 없이 정의한 함수와 같다.
- 컴파일러가 묵시적으로 return Unit 을 넣어주기 때문에 명시적으로 반환하지 않아도 된다.
```kotlin
/**
 * Unit 반환 타입은 반환 타입 선언 없이 정의한 함수와 동일하다.
 */
fun f(): Unit {
    // ...
}

fun f2() {
    // ...
}
```
> 대부분의 경우 void 와 Unit 의 차이를 알기는 어렵다.
> 코틀린 함수 반환 타입이 Unit 타입이고 제네릭 함수를 오버라이드 하지 않는다면, 자바 void 함수로 컴파일 된다.

- 함수형 프로그래밍에서 전통적으로 **Unit 은 단 하나의 인스턴스만 갖는 타입을 의미**해 왔다.
- 그 유일한 인스턴스 유무가 바로 자바 void 와 코틀린 Unit 을 구분하는 큰 차이이다.

#### Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다.
- 코틀린에서 결코 성공적으로 값을 돌려주는 일이 없으므로 반환값 이라는 개념 자체가 의미없는 함수가 일부 존재한다.
- 테스트 라이브러리 의 fail 같은 함수들은 반환 값 없이 예외를 던져 테스트 실패시킨다.
- 이런 함수를 호출하는 코드 분석시 함수가 정상적으로 끝나지 않는다는 사실을 알려주기 위해 Nothing 이라는 반환 타입이 있다.

```kotlin
import java.lang.IllegalStateException

/**
 * 정상적으로 끝나지 않는 함수의 경우 (테스트 프레임워크의 fail등..)
 * Nothing 반환 타입을 명시해 줌으로써 정상적으로 끝나지 않음을 표시해주면 유용하다.
 */
fun fail(message: String): Nothing {
    throw IllegalStateException(message)
}
```

#### 널 가능성과 컬렉션
- 컬렉션 안에 널 값을 넣을수 있는지 여부는 중요하다.
- 타입 인자로 쓰인 타입에도 ? 를 붙이면 Nullable 해진다.

```kotlin
import java.io.BufferedReader
import java.lang.NumberFormatException

/**
 * 널이 될수 있게 만들때는 주의 해야한다
 * List<Int?> 는 내부 요소가 널이 될 수 있고
 * List<Int>? 는 리스트 전체가 널이 될 수 있다.
 */
fun readNumbers(reader: BufferedReader): List<Int?> {
    val result = ArrayList<Int?>()
    // BufferedReader 를 코틀린에서 다루는 베스트한 방법
    for (line in reader.lineSequence()) {
//        try {
//            val number = line.toInt()
//            result.add(number)
//        } catch (e: NumberFormatException) {
//            result.add(null)
//        }
        // 코틀린 1.1 이후
        val number = line.toIntOrNull()
        result.add(number)
    }
    return result
}
```

> List<Int?>? 는 리스트도 널이 될 수 있고, 리스트 내부 요소도 널이 될 수 있다.
