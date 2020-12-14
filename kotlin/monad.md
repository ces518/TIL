# 모나드

## 모나드란 ?
- 수학의 범주론에서 사용되는 구조
- 범주론에서 모나드는 내부 함자 범주의 모노이드 대상
- flatMap 은 그저 flat 하게 만들어주는 map 이구나 (잘못된 이해...)
- 비동기 연산 처리시 Promise 도 모나드의 일종
- Null 처리 대신 Optional 을 사용할 수 있음. 이도 모나드의 일종이다.

## 모나드의 정의
- 값을 담는 컨테이너의 일종
- Functor 를 기반으로 구현되었다.
- flatMap() 메소드를 제공한다.
- Monad Laws 를 만족시키는 구현체를 말한다.

### Functor 란 ?
```java
interface Functor<T> {
    <R> Functor<R> map(Function<T, R> f);
}
```
- 함수를 인자로 받는 map 메소드 하나만 가져아한다.
- T 값을 가지는 컨테이너이다.
- T 값을 받아 R 타이블 반환하는 함수
- Functor 는 map 함수를 거치는 R 타입의 Functor 이다.

### map 의 진정한 의미
- 컬렉션의 원소를 순회하는 방법이 아니다.
- T 타입 Functor 를 R 타입 Functor 로 바꾸는 기능

### Functor 는 왜쓰는걸까 ?
- 값을 꺼낼수도 없고 map 으로 값을 변경하는 일만하는데 왜쓸까 ?
- 일반적으로 모델링 할수 없는 상황을 모델링할 수 있다.
    - 값이 없거나, 미래에 준비될 것으로 예상되는 케이스
- 함수를 손쉽게 합성할 수 있다.

### Functor - 값이 없는 경우
- 사용하는 쪽에서 null check 가 불필요하다.
- null 인 경우 로직이 실행되지 않음
- 타입 안정성을 유지하며 null 을 인코딩 하는 방법
```java
// 널인경우
Optional<String> optionalStr = Optional.ofNullable(null)
Optional<String> optionalInt = optionalStr.map(Integer:parseInt)

// 널이 아닌경우
Optional<String> optionalStr = Optional.ofNullable("1")
Optional<String> optionalInt = optionalStr.map(Integer:parseInt)
```
> 널인 경우와 널이 아닌경우의 로직이 100% 동일하다.

### Functor - 값이 미래에 준비되는 케이스
- 미래에 준비되지만, 마치 값이 있는것처럼 처리할 수 있음.
```java
Promise<Customer> customer = // ...
customer.map(Customer::getAddress)
    .map(Address:street)
    .map(String::toLowerCase)
```
> 비동기로 동작하지만 마치 동기로 동작하는것처럼 작성할 수 있음.
> 비동기 연산들의 합성이 가능하다.

List 도 일종의 Functor 이다.
그저 Functor가 List 를 담고 있는것...
리스트의 모든 원소에 함수 f 를 적용하는 것이다.


### 그래서 Monad 가 뭔데 ?
- Monad 는 Functor 에 flatMap() 을 추가한것이다.
- Functor 에 문제가 있어서 나온것.
- Functor 가 Functor 에 감싸져있으면, 함수 합성과 체이닝을 저해한다..

### flatMap 의 진정한 의미
```java
interface Monad<T, M extends Monad<?, ?>> extends Functor<T,M> {
    M flatMap<Function<T, M> f);
}
```
> Map 과의 차이는 M 을 그대로 반환한다.

### 모나드의 의의
- 값이 없거나, 미래에 사용가능할 상황 등 일반적으로 할수 없는 여러 상황을 모델링 할 수 있음
- 비동기 로직을 동기 로직을 구현하는것과 동일한 형태로 구현하면서도 함수의 합성 및 완전한 논블러킹 파이프라인을 구현할 수 있다.

## 참고
- https://www.youtube.com/watch?v=jI4aMyqvpfQ