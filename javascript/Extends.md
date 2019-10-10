# 상속

```javascript
function Car(name,speed){
    this.name = name;
    this.speed = speed;
}
Car.prototype.move = function(){
    console.log(this.name + 'speed = ' + this.speed);
}

var tico = new Car('tico',50);
tico.move();

function FireCar(name,speed,water){
    Car.apply(this,arguments);
    this.water = water;
}

FireCar.prototype = Object.create(Car.prototype);
FireCar.prototype.constructor = FireCar;
FireCar.prototype.splash = function(){
    console.log('Water = ' + this.water + 'L');
}
```

> FireCar 는 Car를 상속한다.
Car.apply(this,arguments); 는 Car의 this들을 그대로 받으라는 의미이다.
FireCar의 매개변수들을 Car의 생성자를 통해 전달하고 Car의 this들을 FireCar의 속성으로 적용한다.

FireCar.prototype = Object.create(Car.prototype); 은 Car의 prototype을 상속받는 객체를 만들어 prototype으로 지정한다

* Object.create(Car.prototype); 와 new Car() 의 차이 
- Object.create 는 객체를 만들되 생성자는 실행하지않는다 , 즉 프로토타입만 넣는것

* FireCar.prototype.constructor = FireCar; 
- 이 코드가 없다면 FireCar.prototype.constructor === Car ; 가 되므로 
FireCar.prototype.constructor = FireCar; 로 오류를 수정해주는 코드이다.
(생성자.prototype.constructor === 생성자 이여야한다.)

