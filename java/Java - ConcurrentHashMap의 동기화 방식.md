#### Java - ConcurrentHashMap의 동기화 방식
- JDK 1.8 기준
- 기본적인 동작은 HashMap 과 동일하거나 비슷한 부분이 많다.

#### 동기화 처리 방식
- Hashtable과 다르게 주요 method에 synchronized 키워드가 선언되어 있지 않다.

- 1. 빈 해시버킷에 노드를 삽입하는 경우, lock 을 사용하지 않고 Compare and Swap을 이용하여 새로운 노드를 해시 버킷에 삽입한다.
- 2. 이미 노드가 존재하는 경우에는 synchronized 를 이용해 하나의 스레드만 접근 할 수 있도록 제어한다.

> 서로 다른 스레드가 같은 해시 버킷에 접근할 때만 해당 블록이 잠기게 된다.
