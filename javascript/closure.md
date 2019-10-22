# 클로저

- IIFE 가 클로저를 활용한 패턴이다.
- IIFE가 모두 클로저인게 아니다.
- 비공개 변수를 가질수 있는 환경에있는 함수가 클로저이다.
- 비공개 변수는 클로저 함수내부에 생성한 변수도아니고,
매개 변수도 아니다.
- 클로저를 말할땐  '스코프,컨텍스트,비공개 변수와 함수의 관계' 를 
항상 같이말해주어야한다.

```javascript
var makeClosure = function() {
  var name = 'zero';
  return function () {
    console.log(name);
  }
};
var closure = makeClosure(); // function () { console.log(name); }
closure(); // 'zero';
```

