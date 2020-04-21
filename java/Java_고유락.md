#### Java의 고유락(intrinsic lock)

##### 고유 락
- 자바의 모든 객체는 락(lock)을 가지고 있다.
- 고유락, 혹은 모니터락이라고도 한다.


##### synchronized 블록
- 자바의 synchronized 블록은 동시성 문제를 해결하는 가장 간편한 방법이다.
- 고유 락을 이용하여 여러 스레드의 접근을 제어한다.

```java
class Counter {
    private int count;

    public synchronized int increase() {
        return ++count;
    }
}
```

##### 재진입 가능성 (Reentrancy)
- 자바의 고유 락은 재진입이 가능하다.
- 락의 획득이 호출 단위가 아닌 스레드 단위로 일어난다는 것을 의미한다.
- 이미 락을 획득한 스레드는 락을 얻기 위해 대기할 필요가 없다.

```java
class Reentrancy {
  public synchronized void a() {
    System.out.println("a");
    // b가 synchronized로 선언되어 있지만 a진입시 이미 락을 획득하였으므로,
    // b를 호출할 수 있다.
    b();
  }
  public synchronized void b() {
    System.out.println("b");
  }
}
```

> 자바의 고유 락이 재진입을 허용하지 않는다면 위의 코드는 데드락이 발생한다.

##### 구조적인 락 (structured lock)
- 고유 락을 이용한 동기화를 구조적인 락(structured lock) 이라고 한다.
- synchronized 블록 단위로 락의 획득/해제가 일어나므로 구조적이라고 표현한다.

>  A 획득 -> B 획득 -> B해제 -> A해제는 가능하지만 A 획득 -> B획득 -> A해제 -> B해제는 불가능 하다. 이런 경우 ReentrantLock과 같은 명시적인 락을 사용해야 한다.

##### 가시성 (visibility)
- 동시성 이슈중 하나는 가시성 이다.

`가시성 문제란 ?`
- 여러 스레드가 동시에 한 변수에 접근하는 일이 없다고 하더라도 한 스레드가 쓴 값을 다른 스레드가 볼 수도 있고 그렇지 않을수도 있는 문제 이다.
- 문제의 원인은 최적화를 위해 컴파일러나 CPU에서 발생하는 코드 재배열, 멀티 코어 환경에서 코어의 캐시 값이 제때 쓰이지 않아 문제가 발생할 수 있다.

> 자바에서는 스레드가 락을 획득하는 경우 그 이전에 쓰였던 값들의 가시성을 보장한다.
