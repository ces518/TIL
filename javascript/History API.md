# History API
- javascript 에서 비동기 통신 (AJAX) 방식을 이용한 데이터 처리시 새로고침을 하면 이전 페이지가 저장되지 않는 문제가 있다.
- 이를 history api 를 사용해 history를 추가함으로써 해결할 수 있다.

history.pushState(state, title, url); 함수 활용
```javascript
history.pushState(null, null, "/test/test.do?name=abc");
```
- state : state객체, 640kb 크기 제한(over 시 예외발생)
    - 데이터를 저장할수 있어 유용하다
- title : Firefox 는 무시함, 빈문자열을 보내는 것을 권장, 짧은 명칭 부여 가능
    - 브라우저의 제목
- url : history 에 등록할 url
    - 바뀔 주소

* pushState는 hashChange이벤트를 발생시키지 않는다.
- history에만 추가할뿐 아무런 동작을 하지 않음
- 유사한 함수로 *replaceState()* 가 존재한다.

- pushState는 history에 남아 뒤로가기가 활성화 되지만, replaceState는 뒤로가기가 활성화 되지 않는다.

- 참고
    - https://developer.mozilla.org/ko/docs/Web/API/History_API
    - https://www.zerocho.com/category/HTML&DOM/post/599d2fb635814200189fe1a7
    - https://cofs.tistory.com/401
