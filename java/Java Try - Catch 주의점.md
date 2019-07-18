# Java Try - Catch 주의점

- ContentsNotFoundException.java
```java
public class ContentsNotFoundException extends RuntimeException {
  ... 
}
```

- try-catch 를 사용할경우 catch부분에는 로직을 넣지 않는 것이 좋다.
  - 로직을 넣을 경우 무한루프의 가능성이 높기 때문에 Error메시지 정도만 출력해 준다.
- catch로 예외처리를 할 경우 상위 계층 일수록 아래쪽에 써준다.
- Try-Catch-Finally을 쓸때는 블럭변수로 선언되지 않도록 주의한다.

```java
try {

} catch (ContentsNotFoundException e) {
  ...
} catch (RuntimeException e) {
  ...
} catch (Exception e) {
  ... 
}
```
