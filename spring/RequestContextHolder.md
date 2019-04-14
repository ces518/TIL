# RequestContextHolder
- Spring에는 매개변수로 HttpServletRequest를 받지않아도 현재 요청 스레드에서 HttpServletRequest를 가져올수 있는 유틸클래스가 존재한다.
- 해당 유틸클래스에는 메서드가 두개 존재하는데
- getRequestAttributes() 와 currentRequestAttributes() 이다.
- 모두 현재 요청 스레드에서 Request 객체를 가져오는것은 동일하나 getRequestAttributes는 없을시 null을 리턴하고
- currentRequestAttributes 는 예외를 발생시킨다는 차이점이 존재한다.

```java
ServletRequestAttributes servletRequestAttribute = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
servletRequestAttribute.getRequest();
```
