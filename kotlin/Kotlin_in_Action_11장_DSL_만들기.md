# Kotlin in Action

## 11장 DSL 만들기
- DSL (Domain-Specific Language) 를 사용해 코틀린스러운 API 를 방법
- 전통 API 와 DSL API 의 차이
- 코틀린 DSL 설계는 코틀린 언어의 특성들을 활용한다.

#### API 에서 DSL 로
- 궁극적인 목표는 코드의 가독성과 유지보수성을 좋게 유지하는 것
- 개별 클래스에 집중하는것 만으로는 쉽지 않다.
- 대부분의 소스코드는 다른 클래스들과 상호작용 하는데, 상호작용은 클래스의 API 를 통해 일어난다.
- 모든 개발자는 API 잘 만들기 위해 노력해야 한다.

#### API 가 깔끔하다 ?
- 코드를 읽는 사람이 어떤 일이 일어날지 명확하게 이해할 수 있어야 한다.
- 코드가 간결하고, 불필요한 구문이나 번잡한 준비코드가 가능한 적어야 한다.

#### 코틀린이 간결한 구문을 지원하는 방법
| 일반 구문 | 간결한 구문 | 언어 특성 |
|---|---|---|
| StringUtils.capitalize(s) | s.capitalize() | 확장 함수 |
| 1.to("one") | 1 to "one" | 중위 호출 |
| set.add(2) | set += 2 | 연산자 오버로딩 |
| map.get("key") | map["key"] | get 메소드에 대한 관례 |
| file.use({ f -> f.read() }) | file.use { it.read() } | 람다를 괄호 밖으로 빼내는 관례 |
| sb.append("yes") | with (sb) { append("yes") } | 수신 객체 지정 람다 |

#### 영역 특화 언어
- DSL 이라는 개념은 오래된 개념이다.
- 컴퓨터 발명 초기부터 **범용 프로그래밍 언어 (general-purpose programming language)** 와 특정 영역에 초점을 맞춘 특화 언어를 구분해 왔다.
- 우리에게 가장 익숙한 DSL 은 SQL 과 정규식이다.
- 범용 프로그래밍 언어는 명령적 인데 반해 DSL은 범용 프로그래밍 언어와 달리 선언적 이라는 점이 중요하다.
- 명령적 언어는 어떤 연산을 완수하기 위한 각 단계를 순서대로 정확히 기술한다.
- 선언적 언어는 원하는 결과를 기술하고, 세부 실행은 실행 엔진에게 맡긴다.

> 실행엔진이 결과를 얻는 과정에서 최적화를 수행하기 때문에 선언적 언어거 더 효율적인 경우가 빈번하다.

- DSL 의 단점은 자체 문법이 존재하기 때문에 다른 언어의 프로그램에 직접 포함 시킬수 없다.
- DSL 프로그램을 별도의 파일 혹은 문자열 리터럴로 저장해야 한다.

> 이런 문제를 해결하며 DSL 의 장점을 살리는 방법으로 internal DSL 이라는 개념이 확산되고 있다.

#### 내부 DSL
- 독립적인 문법을 가진 external DSL 과 반대로 내부 DSL 은 범용 언어로 작성된 프로그램의 일부이며 범용 언어와 동일한 문법을 사용한다.
- 내부 DSL 은 다른 언어가 아닌 DSL 의 핵심 장점을 유지하며 주 언어를 특별한 방법으로 사용하는 것이다.

`external dsl`
```sql
SELECT Country.name, COUNT(Customer.id)
FROM Country
JOIN Customer
ON Country.id = Customer.country_id
GROUP BY Country.name
ORDER BY COUNT(Customer.id) DESC
LIMIT 1
```

`internal dsl`
- 코틀린으로 작성된 DB 프레임워크인 Exposed 프레임워크를 사용한것
```kotlin
(Country join Customer)
    .slice(Country.name, Count(Customer.id))
    .selectAll()
    .groupBy(Country.name)
    .orderBy(Count(Customer.id), isAsc = false)
```

> internal dsl 로 작성된 코드는 결과를 코틀린 객체로 변환하는 노력이 필요없다.

#### DSL 의 구조
- DSL 과 일반 API 사이에 잘 정의된 일반적인 경계는 없다.
- 다른 API 에는 없지만 DSL 에만 존재하는 특징은 구조 혹은 문법이다.
- 일반적인 라이브러리는 여러 함수들로 이루어져 있고, 해당 함수 호출 시퀀스에는 아무런 구조가 없다.
- 호출과 호출 사이에 맥락이 존재하지 않는다. 
- 이런 API 를 명령-질의 API 라고 한다.
- 반면 DSL 의 함수 호출은 DSL 문법에 의해 정해지는 커다란 구조에 속한다. 
- 코틀린 DSL 에서는 보통 람다를 중첩 하거나 메소드 호출을 연쇄하는 방식을 사용한다.
- DSL 구조의 장점은 같은 문맥을 함수 호출시 마다 반복하지 않고 재사용 가능하다는 점이다.

`gradle 에서 코틀린 DSL 의 예`
```gradle
dependencies {
    compile("junit:junit:4.11")
    compile("com.google.inject:guice:4.1.0")
}
```

#### 구조화된 API 구축 - DSL 에서 수신 객체 지정 DSL 사용
- 수신 객체 지정 람다는 구조화된 API 설계시 도움이 되는 강력한 기능이다.
- 구조가 있다는 점은 일반 API 와 DSL을 구분하는 중요한 특성이다.

#### 수신 객체 지정 람다와 확장 함수 타입

`람다를 인자로 받는 buildString 함수`
```kotlin
/**
 * 람다를 인자로 받는 buildString() 함수 정의
 */
fun buildString(
        builderAction: (StringBuilder) -> Unit
): String {
    val sb = StringBuilder()
    builderAction(sb)
    return sb.toString()
}

fun main() {
    val s = buildString {
        it.append(1)
        it.append(2)
    }
    // ...
}
```

> 위 함수의 문제점은 람다 내부에서 it 를 지속적으로 사용해야 하는 불편함이 있다.

`수신객체 지정 람다를 사용한 개선`
```kotlin
/**
 * 수신객체 지정 람다를 이용해 편리성을 높힘 
 */
fun buildStringV2(
        builderAction: StringBuilder.() -> Unit
): String {
    val sb = StringBuilder()
    sb.builderAction()
    return sb.toString()
}

fun main() {
    buildStringV2 { 
        append(2)
        append(3)
    }
}
```

- 수신 객체를 지정한 람다를 인자로 넘기기 때문에 람다 내부에서 it 를 사용하지 않아도 된다.
- it.append () 에서 this.append() 로 바뀐것이지만 this 는 생략이 가능하다.
- buildStringV2 에서 넘긴 람다는 일반 함수 대신 **확장 함수 (extension function type)** 타입을 사용했다.

`수신 객체 지정 확장 함수 타입 정의`
```kotlin
[수신객체]          [반환 타입]
 String.(Int, Int) -> Unit
      [파라미터 타입]
```

#### 왜 확장함수일까 ?
- 확장 함수의 본문에서는 확장 대상 클래스에 정의된 메소드를 마치 해당 클래스 내부에서 호출하듯이 사용할 수 있다.
- 확장 함수 혹은 수신 객체 지정람다 에서는 모두 함수를 호출할 때 수신 객체를 지정해야하고, 함수 본문에서 수신객체를 특별한 수식어 없이 사용할 수 있다.
- 위 두번째 예제에서의 builderAction 은 StringBuilder 클래스 냅주에 정의된 함수가 아니며 확장 함수 타입으로 정의한 것 뿐이다.


- 표준 라이브러리의 buildString 의 구현은 훨씬 더 짧다.
- apply 함수의 인자로 넘기으로 인해 단 한줄로 구현이 가능하다.
```kotlin
fun buildString(builderAction: StringBuilder.() -> Unit) = StringBuilder().apply(builderAction).toString()
```

#### apply, with 함수 다시보기
- apply 함수는 인자로 받은 람다나 함수를 호출하여 자신의 수신객체를 람다 혹은 함수의 묵시적 수신객체로 사용한다.
- apply 함수와 with 함수의 구현을 살펴보자.

`apply / with 함수의 구현`
```kotlin
// apply 는 수신객체 자신을 반환한다.
inline fun <T>T.apply(block: T.() -> Unit): T {
    block()
    return this
}

// with 함수는 람다를 호출한 결과를 반환한다.
inline fun <T, R>with(receiver: T, block: T.() -> R): R = receiver.block()
```

> apply 와 with 함수의 가장 큰 차이는 수신객체를 인자로 받느냐 , 수신객체를 반환하냐의 여부이다.

