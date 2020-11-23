# Kotlin in Action

## 코루틴과 Async / Await
- 코틀린 1.3 부터 코루틴이 표준 라이브러리에 정식 포함 되었다.

#### 코루틴이란 ?
`서브루틴의 이해`
- 서브루틴은 여러 명령을 모아 이름을 부여해 반복 호출가능하게 만든 프로그램의 구성요소이다.
- 이는 함수라고 불린다.
- 객체지향 언어에서 메소드도 서브루틴이라고 할 수 있다.
- 서브루틴 호출시마다 **활성 레코드 (active record)** 가 스택에 할당되며 서브루틴 내부의 로컬 변수 등이 초기화 된다.

> 서브루틴이 반환되고 나면 실행중이던 모든 상태를 잃어버린다. 따라서 서브루틴을 반복적으로 호출하더라도 항상 같은 결과를 얻는다.

`멀티태스킹의 이해`
- 멀티태스킹은 여러작업들을 동시에 수행하는것 처럼 보이거나, 동시에 수행하는 것이다.
- 비선점형이란 멀티태스킹의 각 작업을 수행하는 참여자들의 실행을 운영체제가 관여해서 중단시키고 다른 실행을 하게끔 만들수 없다는 의미이다.
- 각 참여자들이 서로 자발적으로 협력해야 비선점형 멀티테스킹이 제대로 동작한다.

- **코루틴** 이란 서로 협력하여 실행을 주고 받으며 동작하는 여러 서브루틴을 말한다.
- 코루틴의 대표격인 **제너레이터 (generator)** 를 예로 들면, 함수 A가 실행 도중 제너레이터인 코루틴 B를 호출하면 A를 실행중인
- 스레드 에서 코루틴 B가 실행된다. 코루틴 B 실행도중 A에게 양보한다. (yield 명령어를 사용하는 경우가 많다.)
- 일반적인 함수와달리 코루틴은 yield 로 실행을 양보했던 지점부터 실행을 이어 나간다.
- 자바스크립트의 generator 함수

#### 코틀린의 코루틴 지원 - 일반적인 코루틴
- 언어별로 코루틴을 지원하는 방법은 제너레이터 등 특정 형태의 코루틴만을 지원하거나
- 좀 더 일반적인 코루틴을 만들수 있는 기능을 언어가 제공하고 이를 이용해 사용자가 직접 구현 또는 라이브러리를 통해 제공한다.
- 코틀린은 특정 코루틴을 언어가 지원하는 형태가 아닌 코루틴을 구현할 수 있는 기본 도구를 언어가 제공하는 형태이다.
- kotlin.coroutine 패키지 하위에 존재하고, 코틀린 1.3부터는 코틀린을 설치하면 별도 설정없이 사용이 가능하다.

#### 여러가지 코루틴
- kotlinx.coroutines.core 모듈에 존재하는 코루틴 빌더를 사용해 코루틴을 만든다.
- 코틀린에서는 코루틴 빌더에 원하는 동작을 람다로 넘겨 코루틴을 만들어 실행하는 방식을 사용한다.

#### kotlinx.coroutines.CoroutineScope.launch
- launch 는 코루틴을 Job 으로 반환하며, 생성된 코루틴은 즉시 실행된다.
- Job 의 cancel() 함수를 호출해 실행을 중단시킬 수 있다.
- GlobalScope.launch 가 만들어낸 코루틴은 메인 함수와 서로 다른 스레드에서 실행된다.
- 또한 GlobalScope 는 메인 스레드가 실행중인 동안에만 코루틴의 동작을 보장한다.

#### CoroutineScope
- CoroutineScope 는 새로운 코루틴을 생성함과 동시에 실행될 Job 을 그루핑한다.
- 코루틴 컨텍스트에는 Main, IO, Default 세가지 가 있다.
1. Main 은 메인 스레드에 대한 Context 이며 UI 갱신, Toast 등 View 작업에 사용된다.
2. IO 는 네트워킹, DB 접근 등 백그라운드에서 필요한 작업을 수행할 때 사용된다.
3. Default 는 크기가 큰 리스트를 다루거나 필터링을 수행하는 등 무거운 연산이 필요한 작업에 사용된다.

```kotlin
CoroutineScope(Main).launch {
    // do something
}

CoroutineScope(IO).launch {
    // do something
}

CoroutineScope(Default).launch {
    // do something
}
```

#### Suspend
- kotlinx.coroutines.CoroutineScope.launch 는 suspend CoroutineScope.() -> Unit 이다.
- 여기서 suspend 라는 키워드는 처음 접한다.
- suspend 키워드가 붙은 함수는 suspend 함수가 된다.
- suspend 함수는 해당 함수가 비동기 환경에서 사용될 수 있다는 의미를 내포한다.
- 비동기 함수인 suspend 함수는 다른 suspend 함수 혹은 코루틴 내에서만 호출할 수 있으며, 다른 곳에서 호출할 경우 waring 에러가 발생한다.

#### 코루틴 컨텍스트와 디스패처
- launch, async 등은 모두 CoroutineScope 의 **확장 함수** 이다.
- CoroutineScope 에는 CoroutineContext 타입 필드 하나만이 존재한다.
- CoroutineScope 는 CoroutineContext 를 launch 등 함수 내부에서 사용하기 위한 **매개체 역할** 만을 수행한다.
- CoroutineContext 는 실제 코루틴이 실행 중인 여러 Job 과 디스패처를 저장하는 일종의 맵이라 할 수 있다.
- 코틀린 런타임은 CoroutineContext 를 사용해 다음 실행 작업을 선정하고, 어떻게 스레드에 배정할지에 대한 방법을 결정한다.

#### 코루틴 빌더와 일시 중단 함수
- launch, async, runBlocking 등은 모두 코루틴 비럳라고 불린다.
- 이들은 모두 코루틴을 생성해주는 함수이다.
- kotlinx-coroutines-core 모듈이 제공하는 코루틴 빌더는 아래 두가지 종류가 더 있다.
1. produce
    - 정해진 채널로 데이터 스트림을 보내는 코루틴을 생성한다.
    - ReceiveChannel<> 을 반환한다.
    - 해당 채널로 부터 메시지를 전달받아 사용이 가능하다.
2. actor
    - 정해진 채널로 메시지를 받아 처리하는 액터를 코루틴으로 생성한다.
    - SendChannel<> 을 반환한다.
    - send() 메소드를 통해 액터에게 메시지를 보낼 수 있다.
- delay(), yield() 함수는 코루틴 내에서 특별한 의미를 지니는 함수들이다.
- 이들은 **일시 중단 (suspending) 함수** 라고부른다.
- 이외에도 kotlinx-coroutines-core 모듈에 정의된 일시 중단 함수들은 다음과 같다.
1. withContext
    - 다른 컨텍스트로 코루틴을 전환한다.
2. withTimeout
    - 코루틴이 정해진 시간안에 실행되지 않으면 예외를 발생시킨다.
3. withTimeoutOrNull
    - 코루틴이 정해진 시간에 실행되지 않으면 null 을 리턴한다.
4. awaitAll
    - 모든 작업의 성공을 기다린다.
    - 작업중 하나가 예외로 실패하면 awaitAll 도 예외로 실패한다.
5. joinAll
    - 모든 작업이 끝날 때까지 현재 작업을 일시 중단시킨다.

# 참고
- https://blog.yena.io/studynote/2020/04/26/Android-Kotlin-Coroutine.html