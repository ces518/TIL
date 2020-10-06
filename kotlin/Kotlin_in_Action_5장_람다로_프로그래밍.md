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
