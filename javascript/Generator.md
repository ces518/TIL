# Generator

- Generator란 ? 함수의 실행을 제어할수있는 함수이다. 
- 일반적인 함수의 경우에는 함수를 호출하는순간 함수의 로직이 모두 실행되며 제어를 할수가 없다.

```javascript
function normal () {
  console.log(1);
  console.log(2);
  console.log(3);
  console.log(4);
}

normal();
// 1,2,3,4

function* generator() {
  yield console.log(1);
  yield console.log(2);
  yield console.log(3);
  yield console.log(4);
}

const gen = generator();
gen.next(); //1
gen.next(); //2
gen.next(); //3
gen.next(); //4
```

- generator의 경우에는 함수의 실행을 제어할수 있으며 next() 함수 호출을통해 실행을 제어한다.
- yield 키워드가 사용되는데 yield키워드가 그 실행의 종단점이라고 생각하면된다.
- yield console.log(1); 부분이 next()를 호출하면 실행되는 종단지점이고 그다음 yield를 만나면 실행을 멈추게된다.
- 실행 종단점마다 값을 return할수도 있다.
- {value: returnValue, done: false} 의 형태로 리턴되며,
- 더이상 함수의 종단점이 존재하지않고, 함수의 끝 블록을 만난경우엔 genetator함수도 종료되며 next()를 호출해도 더이상 실행되지않는다.


- Redux-SAGA eventListener 와 같은 역할로 사용한다.

### 반복문
- generator에서는 반복문도 제어가 가능해진다.
- 앞서 말한것처럼 generator가 함수의 실행을 제어할수 있기때문에 다음과 같이 경이로운 문법이 가능해진다.
```javascript
function* genetaor () {
  while (true) {
    yield 1;
  }
}

const gen = generator();
gen.next(); //1
gen.next(); //1
```

- while (true) 로 무한반복문이되어 애플리케이션이 멈출것같지만 yield라는 종단점을 만날때마다 함수의 실행이 중단되며 next()호출을 대기하기때문에
- 무한루프에 빠지지않게된다.
- 해당 문법을 활용하여 Redux-SAGA에서 eventListener와 같은 역할을 하게된다.
- 해당 부분을 조금 응용하게 되면 다음과 같은 처리도 가능해진다.


```javascript
function* fiveStepClick () {
  for (let i = 0; i < 5; i++) {
    yield 1;
  }
}

const gen = generator();
gen.next(); //1
```

- 특정 이벤트의 처리를 5번만 허용하고싶을경우 다음과 같이 활용하여 제어가 가능해진다.
