# 스코프 체인	

- 전역변수와 지역변수의 관계에서 스코프체인 이라는 개념이나온다.	

```javascript
var name = 'zero';	
function outer() {	
  console.log('외부', name);	
  function inner() {	
    var enemy = 'nero';	
    console.log('내부', name);	
  }	
  inner();	
}	
outer();	
```
1. inner함수는 name 변수를 찾기위해 먼저 자신의 스코프에서 찾는다.	
2. 자신의 스코프에 없다면 한 단계 올라가 outer스코프에서 찾는다.	
3. outer스코프에도 없다면 전역 스코프에서 찾는다.	

* 이렇게 꼬리를 물고 계속 범위를 넓히면서 찾는관계를 스코프체인 이라고한다.
