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