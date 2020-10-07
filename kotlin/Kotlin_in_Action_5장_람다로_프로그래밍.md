# Kotlin in Action

## 5장 람다로 프로그래밍
- 람다 식 (lamda expression) 은 기본적으로 다른 함수에 넘길 수 있는 **작은 코드 조각** 이다.
- 람다를 사용하면 공통 코드 구조를 라이브러리화 하기 쉽다.
- 코틀린 표준 라이브러리는 람다를 아주 많이 사용한다.

#### 람다 소개 - 코드 블록을 함수 인자로 넘기기
- 자바에서는 익명 내부 클래스를 통해 코드를 함수에 넘기거나 변수에 저장했다. 
    - 클래스를 선언하고 클래스 인스턴스를 함수에 넘기는 방법을 사용
    - 이는 매우 번거로운 일이다..
- 함수형 프로그래밍 에서는 함수를 값처럼 다룰 수 있다.
    - 함수를 직접 다른 함수에 전달할 수 있다.
    
`java`
```java
button.setOnClickListener(new OnClickLisntener() {
    @Override
    public void onClick(View view) {
        /* do Something */
    }
})
```

`kotlin`
```kotlin
button.setOnClickListener { /* do Something*/ }
```

#### 람다와 컬렉션
- 코드 중복을 제거하는 것은 프로그래밍 스타일을 개선하는 중요한 방법중 하나
- 컬렉션을 다룰때 수행하는 대부분의 작업은 몇 가지 일반적인 패턴이다.
- 이는 라이브러리 내에 존재해야한다.

```kotlin
data class SamplePerson(val name: String, val age: Int)

/**
 * 가장 연장자를 찾는 함수
 * 이는 코드가 장황하고 실수를 저지르기 쉽다.
 */
fun findTheOldest(people: List<SamplePerson>) {
    var maxAge = 0
    var theOldest: SamplePerson? = null
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest)
}

/**
 * 람다를 사용해 해결
 */
fun main(args: Array<String>) {
    val people = listOf(SamplePerson("ncucu", 27), SamplePerson("ncucu.me", 28))
    println(people.maxBy { it.age })

    // 멤버 참조를 사용해 해결
    println(people.maxBy(SamplePerson::age))
}
```

> 람다 표현식 내에서 함수나, 프로퍼티를 반호나하는 역할을 수행하는 경우 멤버 참조로 코드를 좀 더 간결하게 할 수 있다.

#### 람다식의 문법
- 람다는 값처럼 여기저기 전달할 수 있는 코드 조각이다.
- 변수에 저장할 수 있지만, 대부분 함수에 인자로 넘기면서 바로 람다를 정의하는 경우가 대부분이다.
- 코틀린의 람다식은 항상 중괄호로 둘러 쌓여 있으며, 인자 목록 주변에 괄호가 없다.
```kotlin
{ x: Int, y: Int -> x + y }
[    파라미터    ] [ 본문 ]
```
- 코드의 일부분을 둘러싸 실행할 필요가 있다면 run 을 사용한다.
    - run 은 인자로 받은 람다를 실행해주는 함수이다.
```kotlin
run { println(42) }
```

> 실행 시점에 코틀린 람다는 부가 비용이 없으며, 프로그램 기본 구성요소와 비슷한 성능을 낸다.

- 코틀린에서는 함수 호출 시 맨 뒤에 있는 인자가 람다 식이라면, 람다를 괄호 밖으로 빼낼 수 있다.
```kotlin
// 정식 람다 호출
people.maxBy({ p: Person -> p.age })
// 마지막 인자가 람다인경우 
people.maxBy() { p: Person -> p.age }
// 어떤 함수의 유일한 인자이고, 괄호 뒤에 람다를 사용했다면, 호출 시 빈괄호를 제거할 수 있다.
people.maxBy { p: Person -> p.age }
```

- 로컬 변수처럼 람다 파라미터 타입도 추론이 가능하다.
    - 파라미터 타입 명시가 필요 없다.
- 람다의 파라미터가 하나 이고, 컴파일러가 타입추론이 가능한경우 it 라고 하는 변수로 접근할 수 있다.
    - 이는 코드를 간결하게 해주지만 남용해서는 안된다.
    - 람다 내에 람다가 중첩되는 경우 람다의 파라미터를 명시하는 편이 좋다.
```kotlin
people.maxBy { p -> p.age }

people.maxBy { it.age }
```

> 본문이 여러 줄로 이루어져 있는 경우 본문의 *맨 마지막 식*이 람다의 결과가 된다.

#### 현재 영역에 있는 변수에 접근
- 자바 메소드 내에서 익명 내부 클래스 정의시 메소드 로컬 수를 익명 내부 클래스에서 사용할 수 있다.
- 람다에서도 동일하게 함수 내에서 정의한다면 함수 파라미터 뿐이 아닌 람다 앞에 존재하는 로컬 변수까지 모두 사용이 가능하다.
- 람다 내에서 사용하는 외부 변수를 람다가 포획한 변수 라고 한다.
```kotlin
/**
 * 람다 내에서 함수 파라메터에 접근이 가능하다.
 * 자바와 다른 점이라면, final 변수가 아닌 변수에도 접근할 수 있다.
 * 람다 안에서 사용하는 외부 변수를 람다가 포획한 변수 라고 부른다.
 */
fun printMessageWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it")
    }
}

fun main(args: Array<String>) {
    val errors = listOf("403 Forbidden", "404 Not Found")
    printMessageWithPrefix(errors, "Error: ")
}

fun printProblemCounts(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0
    responses.forEach {
        if (it.startsWith("4")) {
            clientErrors++
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }
    println("$clientErrors client errors, $serverErrors server errors")
}

fun main2(args: Array<String>) {
    val responses = listOf("200 OK", "418 I'm a teapot", "500 Internal Server Error")
    printProblemCounts(responses)
}
```

> 람다를 실행 시점에 표현하는 데이터 구조는 람다에서 시작한 모든 참조가 닫힌 객체 그래프를 람다 코드와 함께 저장해야 한다.
> 이런 구조를 클로저 라고 한다.
> 함수를 1급 시민으로 만드려면 포획한 변수를 잘 처리하기 위해 클로저가 꼭 필요하다.

- 어떤 함수가 로컬변수를 포획한 람다를 반환하거나, 다른 변수에 저장한다면 로컬 변수의 생명주기와 함수의 생명주기가 달라질 수 있다.
- 함수가 끝난뒤에도 람다의 본문 코드는 포획한 변수를 읽거나 쓸 수 있다. (이런 구조가 클로저)

#### 멤버 참조
- 코틀린에서는 자바8과 마찬가지로 함수를 값으로 바꿀수 있다.
- 이중 콜론 (::) 을 사용한다.
- 이런 방법을 **멤버 참조 (member reference)** 라고 한다.
- 함수 언어에서는 이런 경우를 에타 변환이라고 한다.
    - 에타 변환이란 함수 f와 람다 { x -> f(x) } 를 서로 바꿔쓰는것을 의미한다.
    
```kotlin
/**
 * 멤버 참조
 */
fun main(args: Array<String>) {
    // 멤버 참조는 아래의 람다식을 심플하게 표현한것이다.
    var getAge = SamplePerson::age
    val getAge2 = { person:SamplePerson -> person.age }

    val people = listOf(SamplePerson("ncucu", 27), SamplePerson("ncucu.me", 27))
    people.maxBy(SamplePerson::age)

    // 최상위에 선언된 함수나 프로퍼티를 참조할 수 있다.
    // 이 경우에는 클래스 명을 생략하고 :: 를 시작한다.
    fun hello() = println("Hello")
    run(::hello)
}
```

- 생성자 참조를 사용하면, 클래스 생성 작업을 연기하거나 저장해 둘 수 있다.
- ::ClassName 형태로 생성자 참조를 만들 수 있다.

```kotlin
/**
 * 생성자 참조를 사용하면 클래스 생성 작업을 연기하거나, 저장해 둘 수 있다.
 * ::클래스명 형태로 생성자 참조를 만들 수 있다.
 */
fun main(args: Array<String>) {
    val createSamplePerson = ::SamplePerson
    val p = createSamplePerson("ncucu", 27)
    println(p)
}
```

> 확장 함수도 멤버 함수와 동일한 방식으로 참조가 가능하다.

#### 바운드 멤버 참조
- 코틀린 1.1 부터 바운드 멤버 참조를 지원한다.
- 멤버 참조를 생성할 때 클래스 인스턴스를 함께 저장한 뒤 나중에 그 인스턴스에 대해 멤버를 호출해 준다.
```kotlin
/**
 * 바운드 멤버 참조
 */
fun main(args: Array<String>) {
    val p = SamplePerson("ncucu", 27)
    val personAgeFunction = SamplePerson::age
    println(personAgeFunction(p))

    // 바운드 멤버참조, 인스턴스를 전달하지 않아도 된다.
    val dmitryAgeFunction = p::age
    println(dmitryAgeFunction())
}
```

#### 컬렉션 함수형 API
- 함수형 프로그래밍 스타일을 사용하면 컬렉션을 다룰 때 매우 편리하다.

#### 필수적인 함수 - filter, map
- filter와 map은 컬렉션 호라용시 기반이 되는 함수이다.
- 대부분의 컬렉션 연산을 두 함수를 통해 표현이 가능하다.

```kotlin
/**
 * filter 와 map
 */
fun main(args: Array<String>) {
    val list = listOf(1, 2, 3, 4)
    val filteredList = list.filter { it % 2 == 0 } // 짝수만 필터링

    val mapedList = list.map { it * it } // 제곱으로 볁환
}
```

> 맵의 경우 키와 값을 처리하는 함수가 별도로 존재한다.
> filterKeys, mapKeys와 filterValues, mapValues

#### all, any, count, find - 컬렉션에 술어 적용
- count 함수는 조건을 만족하는 원소의 수를 반환하고, find는 조건을 만족하는 첫 원소를 반환한다.
`all, any`
```kotlin
/**
 * all 은 모든 원소가 해당 조건을 만족하는지 판단하고
 * any는 해당 조건을 만족하는 원소가 하나라도 있는지 판단한다.
 */
fun main(args: Array<String>) {
    val canBeInClub27 = { p: SamplePerson -> p.age <= 27}

    val people = listOf(SamplePerson("ncucu", 27), SamplePerson("ncucu.me", 20))
    people.all(canBeInClub27) // false

    people.any(canBeInClub27) // true
}
```

`count`
```kotlin
/**
 * Count는 조건을 만족하는 원소의 수를 반환한다.
 */
fun main(args: Array<String>) {
    val canBeInClub27 = { p: SamplePerson -> p.age <= 27}

    val people = listOf(SamplePerson("ncucu", 27), SamplePerson("ncucu.me", 20))
    people.count(canBeInClub27) // 1
}
```

`find`
```kotlin
/**
 * find 는 조건을 만족하는 첫번째 원소를 반환한다.
 */
fun main(args: Array<String>) {
    val canBeInClub27 = { p: SamplePerson -> p.age <= 27}

    val people = listOf(SamplePerson("ncucu", 27), SamplePerson("ncucu.me", 20))
    people.find(canBeInClub27) // name = ncucu, age = 27
}
```

#### count와 size
- count를 사용하지 않고, 컬렉션을 필터링한 결과의 크기를 가져올 경우
- 조건을 만족하는 모든 원소가 들어가는 컬렉션이 생성된다.
- 반면 count는 그런 과정이 없이 원소의 수만 반환하게 때문에 count가 효율적이다.

#### groupBy - 리스트를 여러 그룹으로 이뤄진 맵으로 변경
- 컬렉션의 모든 원소를 특성에 따라 여러 그룹으로 나눌때 사용한다.

```kotlin
/**
 * group by는 어떤 특정 조건에 맞게 그룹을 나눌때 사용한다.
 * 반환값은 LinkedMap이다.
 */
fun main(args: Array<String>) {
    val people = listOf(SamplePerson("A", 27), SamplePerson("B", 20), SamplePerson("C", 20))
    val groupBy = people.groupBy { it.age }
    println(groupBy) // {27=[SamplePerson(name=A, age=27)], 20=[SamplePerson(name=B, age=20), SamplePerson(name=C, age=20)]}
}
```

#### flatMap과 flatten - 중첩된 컬렉션 내의 원소 처리
- flatMap 함수는 인자로 주어진 람다를 컬렉션의 모든 객체에 적용하고
- 람다를 적용한 결과 얻어지는 리스트를 한 리스트로 모은다.

```kotlin
/**
 * flatMap 은 람다를 적용한 결과 얻어지는 리스트를 한 리스트로 모은다.
 */
class Book(val title: String, val authors: List<String>)

fun main(args: Array<String>) {
    val books = listOf(Book("titleA", listOf("ncucu", "ncucume")), Book("titleB", listOf("ncucu1", "ncucu2")))
    books.flatMap { it.authors }.toSet()// books 컬렉션 내에 존재한 모든 저자들의 집합을 Set으로 모은다.
}
```

- 중첩 리스트구조의 리스트를 단지 펴기만 한다면, flatten 함수를 사용하면 된다.
```kotlin
/**
 * 리스트[리스트] 구조를 평평하게 펴기만 해야할때 flatten 을 사용한다.
 */
fun main(args: Array<String>) {
    val books = listOf(listOf("ncucu", "ncucume"), listOf("ncucu1", "ncucu2"))
    books.flatten()
}
```

#### 지연 계산 (lazy) 컬렉션 연산
- map 이나 filter 같은 컬렉션 함수는 결과 컬렉션을 즉시 생성한다.
- 컬렉션 함수를 연 하면 매 단계마다 중간 결과를 새 컬렉션에 임시로 담는다. (중간 처리과정이 필요하기 때문에 매번 메모리를 소비함)
- **시퀀스 (sequence)** 를 사용하면 중간 임시컬렉션을 사용하지 않고 컬렉션 연산을 연쇄한다.
    - 코틀린 지연연산 시퀀스는 Sequence 인터페이스에서 시작한다.
    - 이는 단지 한번에 하나씩 열거 될 수 있는 원소의 시퀀스를 표현한다.
    - 인터페이스 내에 존재하는 iterator 라는 메소드를 통해 시퀀스로부터 원소값을 얻는다.
- asSequence() 확장함수를 호출하면 어떤 컬렉션이든 시퀀스로 변환이 가능하다.
- toList() 등으로 변환을 해주거나 최종 시퀀스의 원소를 하나씩 이터레이션 해야지만 계산을 실행한다.
```kotlin
/**
 * asSequence 를 이용한 지연 연산을 사용하면 컬렉션 매 연산마다 중간 컬렉션이 생성되지 않는다.
 */
fun main(args: Array<String>) {
    val people = listOf(SamplePerson("ncucu", 27), SamplePerson("ncucume", 27))
    // 아래 코드는 연쇄 호출이 2개의 리스트를 만든다.(map의 결과, filter의 결과)
    people.map(SamplePerson::name).filter { it.startsWith("A") }

    // 아래 코드는 1개의 리스트만을 만든다.
    people.asSequence()
            .map(SamplePerson::name)
            .filter { it.startsWith("A") }
            .toList()
}
```

> 큰 컬렉션에 대해 연산을 연산시킬경우 시퀀스를 사용할것을 권장한다.

#### 시퀀스 연산 실행 - 중간 연선과 최종 연산
- 시퀀스 연산은 중간 (intermediate) 연산과 최종 (terminal) 연산으로 나뉜다.
- 중간 연산은 다른 시퀀스를 반환하며, 연산을 연쇄할 수 있다.
- 최종 연산은 결과를 반환한다.

```kotlin
        [        중간 연산        ][ 최종연산 ]
sequence.map { ... }.filter { ... }.toList()
```

- 위의 코드에 경우 컬렉션이라면 모든 원소에 대해 map 을 한 후 filter 를 수행하지만
- 지연 연산의 경우 첫번째 원소에 대해 map, filter 까지 수행한후 다음 원소에 대한 처리를 시작한다. 

#### 자바 스트림과 코틀린 시퀀스
- 자바8 스트림은 시퀀스와 개념이 같다.
- 자바 8을 채택하면 코틀린 컬레션과 시퀀스에서 제공하지 않는 병렬 스트림 연산 기능을 사용할 수 있다.

#### 시퀀스 만들기
- asSequence() 외에도 시퀀스를 만드는 방법은 generateSequence 함수를 사용한다.
- 이는 이전의 원소를 인자로 받아 다음 원소를 계산하는 함수이다.

```kotlin
/**
 * generateSequence 를 사용하면 이전의 원소를 인자로 받아 다음 원소를 계산한다.
 */
fun main(args: Array<String>) {
    val naturalNumbers = generateSequence(0) {  it + 1 }
    val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
    println(numbersTo100.sum()) // 결과는 5050
}
```

> 시퀀스를 사용하는 일반적인 사례는 객체의 조상으로 이루어진 시퀀스를 만드는 것이다.
> 어떤 파일의 상위 디렉터리를 뒤지면서 숨김 속성을 가진 디렉터리를 검사하는 등의 경우에 사용한다.
```kotlin
fun File.isInsideHiddenDirectory() =
    generateSequence(this) { it.parentFile }.any { it.isHidden }

val file = File("/Users/svtk/.HiddenDir/a.txt")
file.isInsideHiddenDirectory() // true
```

#### 자바 함수형 인터페이스 활용
- 코틀린 라이브러리와 람다를 사용하는건 좋지만, 대부분 API는 자바 API 일것이다.
- 코틀린 람다를 자바 API에 사용해도 아무 문제가 없다.
- 코틀린은 함수형 인터페이스를 인자로 받는 자바 메소드 호출시 람다를 넘길수 있게 해준다.

> 코틀린에는 함수타입이 존재하기 때문에, 함수를 인자로 받을경우 함수 타입을 사용해야 한다.
> 코틀린 함수 사용시에는 코틀린 람다를 함수형 인터페이스로 변환해주지 않는다.

#### 자바 메소드에 람다를 인자로 전달
- 함수형 인터페이스를 받는 자바 메소드에 코틀린 람다를 전달할 수 있다.
- 람다와 무명 객체 사이에는 차이가 있다.
- 객체를 명시적으로 선언하는 경우 메소드 호출마다 새로운 객체가 생성된다.
- 람다는 정의가 들어있는 함수의 변수에 접근하지 않는 람다에 대응하는 무명객체 메소드 호출시마다 반복 사용한다.
```kotlin
/**
 * 함수형 인터페이스를 인자로 받는 자바 메소드에 코틀린 람다를 전달할 수 있음.
 */
fun main(args: Array<String>) {
    // 람다를 사용.
    val t = Thread { println("hello") }
    t.start()

    // 무명객체를 명시적으로 선언
    val t2 = Thread(object : Runnable {
        override fun run() {
            println("hello")
        }
    })
    t2.start()
}
```

> 람다가 변수를 포획하면 무명 클래스 내에 포획한 변수를 저장하기 위한 필드가 생기고, 매 호출시마다 무명클래스의 인스턴스를 새로 만든다.
> 하지만 포획하는 변수가 없는 람다라면 인스턴스가 단 하나만 생긴다.

- 컬렉션을 확장한 메소드에 람다를 넘기는 경우 코틀린은 자바메소드를 코틀린에서 호출할 때 쓰는 방식을 사용하지 않는다.
- 코틀린 inline 으로 표시된 함수에게 람다를 넘기면 아무런 무명클래스도 생성되지 않는다.
- 대부분의 코틀린 확장 함수들은 inline이 붙어있다.

> 대부분의 자바 함수형 인터페이스 <=> 람다 변환은 자동으로 이루어진다.

#### SAM 생성자 - 람다를 함수형 인터페이스로 명시적으로 변경
- SAM 생성자는 람다를 함수형 인터페이스의 인스턴스로 변환할 수 있게 컴파일러가 자동으로 생성한 함수이다.
- 컴파일러가 자동으로 변환하지 못하는 경우 SAM 생자를 사용할 수 있다.

```kotlin
/**
 * SAM 생성자 사용
 * 함수형 인터페이스의 인스턴스를 반환하는 메소드가 있다면
 * 람다를 직접 반환할 수 없고, 반환하고 싶은 람다를 SAM 생성자로 감싸야 한다.
 */
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") }
}

fun main(args: Array<String>) {
    createAllDoneRunnable().run()
}
```

- SAM 생성자의 이름은 사용하려는 함수형 인터페이스의 이름과 같다.
- SAM 생성자는 함수형 인터페이스의 메소드 본문에 사용할 람다만 인자로 받아 함수형 인터페이스를 구현하는 클래스의 인스턴스를 반환한다.

> 오버로딩한 메소드 중 어떤 타입의 메소드를 선택해 람다를 변환해 넘겨줘야할지 모호한 경우 명시적으로 SAM 생성자를 적용하면 컴파일 오류를 
> 피할 수 있다.

#### 람다와 this
- 람다는 무명 객체와 달리 인스턴스 자신을 가리키는 **this** 가 없다.
- 컴파일러 입장에서 람다는 그저 코드블록 일 뿐이다.
- 람다 내에서 this 는 람다를 두러싼 클래스의 인스턴스를 가리킨다.

#### 수신 객체 지정 람다 - with 와 apply
- 코틀린 람다의 기능은 수신 객체를 명시하지 않고, 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있다.
- 이를 수신 객체 지정 람다 (lambda with receiver) 라고 부른다.

#### with 함수
- 첫번째 인자로 수신객체를 지정하고, 두번째 인자로 람다를 받는다.
    - 마지막 인자가 람다 이기 때문에() 괄호 밖으로 빼낼수 있다.
```kotlin
/**
 * with는 수신객체를 지정한 람다이다.
 * 첫번째 인자로 받은 객체를 두번째 인자로 받은 람다의 수신객체로 만든다.
 */
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\n Now I Know the alphabet!")
    return result.toString()
}

fun alphabetWith(): String {
    val stringBuilder = StringBuilder()

    return with(stringBuilder) {
        for (letter in 'A'..'Z') {
            append(letter)
        }
        append("\n Now I Know the alphabet!")
        return toString()
    }
}

fun alphabetWithRefactoring(): String = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter) // this.append() 로 스트링빌더 객체를 호출한다. this는 생략할 수 있다.
    }
    append("\n Now I Know the alphabet!")
    toString()
}

fun main(args: Array<String>) {
    println(alphabet())
    println(alphabetWith())
    println(alphabetWithRefactoring())
}
```

#### 수신 객체 지정 람다와 확장 함수 비교
- 확장 함수 내에서 this는 그 함수가 확장하는 타입의 인스턴스를 가리킨다.
- 수신 객체 멤버 호출시 this 를 생략 할 수 있다.
- 어떤 의미에서는 확장 함수를 수신 객체 지정 함수라고 할 수도 있다.

#### apply 함수
- apply 함수는 with 와 같다.
- 유일한 차이는 apply는 항상 자신에게 전달된 객체 (수신된 객체)를 반환한다는 점이다.

```kotlin
import java.lang.StringBuilder

/**
 * With 와 거의 동일하며 항상 자신에게 전달된 객체를 반환한다는 점이 다르다.
 */
fun alphabetApply() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\n Now I Knopw the alphabet!")
}.toString()

fun main(args: Array<String>) {
    println(alphabetApply())
}
```

#### 정리
- 람다를 사용하면 코드 조각을 다른 함수 인자로 넘길수 있다.
- 람다가 함수의 인자인 경우 괄호 밖으로 람다를 뺄 수 있고, 람다의 인자가 하나인 경우 it 라는 디폴트 명으로 참조가 가능하다.
- 람다 안에 있는 코드는 그 람다 외부 함수의 변수를 읽고 쓸 수 있다.
- 메소드, 생성자, 프로퍼티 명 앞에 :: 를 붙이면 멤버 참조가 가능하다.
- filter, map, all, any 등 함수를 활용하면 컬렉션에 대한 대부분의 연산을 직접 원소를 이터레이션 하지 않고 수행할 수 있다.
- 시퀀스를 사용하면 중간 결과를 담는 컬렉션을 생성하지 않는다.
- 함수형 인터페이스를 인자로 받는 자바 함수 호출시 람다를 사용 할 수 있다.
- 수신 객체 지정 람다를 사용하면 람다내에 미리 정해둔 수신 객체의 메소드를 직접 호출할 수 있다.
- 표준 라이브러리의 with 함수를 사용하면 어떤 객체에 대한 참조를 반복해서 객체 메소드 호출이 가능하다.
- apply를 사용하면 어떤 객체라도 빌더 스타일의 API를 사용해 생성하고 초기화할 수 있다.

- https://thdev.tech/kotlin/2020/10/06/kotlin_effective_05/