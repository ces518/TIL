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
