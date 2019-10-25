# Window 객체 

- 브라우저 전체를 담당하는것이 Window 객체이고
웹 사이트만 담당하는것이 Document객체라고 이해하면된다.

Document 객체도 Window객체 내부에 포함된다.

window 객체내의 대표적인 객체들은 

screen, location, history, document 등이 있다.

메서드는 parseInt, isNaN 등이 있다.


* window는 모든객체의 조상이다.
    전역객체 (Global) 이라고 한다. 모든객체를 다 포함하고있기때문에
    window는 생략이 가능하다.
    window 객체 내에는 String,Boolean,Object,Number,Function,Array같은
    자료형도 다 포함된다.

* 선언한 전역변수들은 window 객체내에 등록이된다.




# window.close():
- 현재 창을 닫는다



# window.open();
- 새창을 연다, 팝업의 형태로 열수도있고, 새탭으로도 열수있다.
첫번째 인자로 주소, 두번째인자로 새탭 or 현재탭에 열지 설정할수있으며 
세번째 인자로 각종 설정을 전달할수있다. 
```javascript
open('https://zerocho.herokuapp.com'); // 새 탭
open('https://zerocho.herokuapp.com', '_self'); // 현재 탭
open('', '', 'width=200,height=200'); // 가로세로 200px의 팝업창
```
- 변수.document.wrtie 로 새창의 내용을 변경도 할 수 있다.

팝업창에서 부모창으로 접근이 가능하다. 
opener 객체 활용 



# window.encodeURI() , window.decodeURI() 

- 인코딩,디코딩함수이다.



# window.setTimeout(함수,밀리세컨), window.setInterval(함수,밀리세컨);
```javascript
setTimeout(function() {
  alert('1초 뒤');
}, 1000);

setInterval(function() {
  console.log('1초마다');
});
```

- 프로그래밍시 몇 초 뒤 실행되어야할때 사용한다.
    setTimeout 은 지정한 시간뒤 1회 실행되고
    setInterval 은 지정한 시간마다 반복된다.

```javascript
var timeout = setTimeout(function() {}, 1000);
clearTimeout(timeout);
```
- 실행중 중간에 멈춰야 하는경우, 먼저변수에 지정해둔다음 
그변수를 이용해서 중지할수있다.


# window.getConputedStyle(태그);
```javascript
console.log(getComputedStyle(document.getElementById('app-root')));
```
- 현재적용된 css 속성 값을 알수있다.




# BOM


1. navigator 

navigator.userAgent; // "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36"
- userAgent 정보로 운영체제, 브라우저 정보등을 알수있다.

navigator.language; // "ko"
navigator.cookieEnabled; // true
navigator.vendor; // "Google Inc"


2. screen

- 화면에 대한 정보를 알려준다. 
너비 width, 높이 heigth, 픽셀 pixelDepth, 컬러 colorDepth,화면방향 orientation, 작업표시줄 제외한 너비 availWidth와 높이 availHeight 가 있다.


3. location
```javascript
location.host; // "www.naver.com"
location.hostname; // "www.naver.com"
location.protocol; // "https:"
location.href; // "https://www.naver.com/..."
location.pathname; // "/../.../....."
```

- location 객체는 주소에 대한 정보를 알려준다. (protocol, host, hostname, pathname, href, port, search, hash)
location.reload() 로 새로고침도 가능하다. 
location.replace() 는 현재주소를 다른주소로 교체한다. (페이지 이동이 일어나지만 기록이 남지않는다.)


4. history 
history.forward(), history.go(1) 앞으로 이동 
history.back(), history.go(-1) 뒤로 가기 
history.length 뒤로가기가능한 페이지 개수 

history.pushState(객체, 제목, 주소)
history.replaceState(객체, 제목, 주소)
- 페이지를 이동하지않고 주소만 바꾸어 준다.
객체부분에 페이지에 대한 정보를 추가 할 수 있다.
