# 코틀린의 함수 파라미터가 Immutable 한 이유

```kotlin
fun foo (var x: Int) {
    x = 5
}
```
- 가장 큰 이유는, 코틀린의 함수 파라미터는 pass-by-value 로 동작한다.
- 하지만 함수 파라미터가 mutable 하다면 pass-by-reference 로 동작하는 것 처럼 오해를 할 수 있기 때문임 (런타임시 비용이 많이 듦)
- 또 다른 이유는 primary 생성자에서 혼란을 불러일으킬 수 있기 때문이다.
- 생성자 선언에서 val 또는 var 는 일반적인 함수에서 선언할 때와 다른 것을 의미 (읽기 전용 속성인지, 가변 속성인지 정의) 하기 때문에 혼란을 야기할 수 있다.
- 마지막으로 mutable 한 파라미터는 좋은 스타일이 아니기 때문이다.

## 참고
- https://blog.jetbrains.com/kotlin/2013/02/kotlin-m5-1/