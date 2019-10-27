# 자바스크립트의 this

* this 는 기본적으로 window 객체를 가리킨다.
일반함수 , 콜백함수, 내부함수 = window 
객체의 속성 , 메서드 일경우 = 해당 객체를 가리킨다.

# this가 window가 아닌경우.

1. 객체의 메서드내에서 이용될경우.
```javascript
var obj = {
    a: function(){
        console.log(this);
    }
}
obj.a(); // obj 
```
2. 명시적으로 this 를 바꾸는 함수 
bind , call, apply 를 사용하면 this가 객체를가리킨다.
```javascript
var obj2 = {..};
function b(){
    console.log(this);
}

b(); // window 
b.bind(obj2).call(); // obj2 
b.call(obj2); // obj2 
b.apply(obj2); // obj2 
```
3. 생성자의 경우 
```javascript
function Person(name, age){
    this.name = name;
    this.age = age;
}

Person.prototype.hello = function(){
    console.log(this.name, this.age);
}
// new 로호출하지않을경우엔 this가 window 를 가리킨다. 
```
# JQuery내의 this 
```javascript
$('div').on('click', function() {
  console.log(this); // <div>
  function inner() {
    console.log('inner', this); // inner Window
  }
  inner();
});
```
inner function 의 this는 이벤트리스너가 변경해주지않기때문에
window를 가리킨다.
