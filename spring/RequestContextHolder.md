# RequestContextHolder
- RequestContextHolder 설명 이전에 간단히 HttpServletRequest에 대해 설명

#### HttpServletRequest
- HttpServletRequest는 ServletAPI 이며 요청에 정보에 대한 처리를 Low-Level로 처리할때 사용한다.
- Servlet, Filter 는 물론, Interceptor, AOP, Controller등 사용할 일이 많다.

#### Spring 에서의 HttpServletRequest 
- Java Application에서 HttpServletRequest는 접근 제한이 있다.
- Servlet, Filter, Interceptor, AOP, Controller Level에서만 접근이 가능하다.
- Service, Repository Layer에서 사용하고싶다면 Controller에서 Argument로 전달하는 방식을 사용해야한다.
- 이런 형식의 구현은 권장되는 모양은 아니며, 보통 Vo , Dto 를 만들어 해당 객체를 Service로 전달하는 형태로 구현한다.

#### Spring RequestContextHolder
- Spring 2.x 부터 제공되는 기능으로, Controller, Service, Repository 전 구간에서 HttpServletRequest에 접근할수 있도록 해주는 객체이다.
- Spring 은 특정 Context를 생성하여, Servlet이 호출되면 key, value 형태로 HttpServletRequest 객체를 저장해 두었다가, Servlet종료시, 해당 Context에서 저장되어있던 HttpServletRequest객체를 제거하는 방식이다.

- Spring 에서 RequestContextHolder를 활용하여 HttpServletRequest에 접근하는 방법 
```java
HttpServletRequest req = ((ServletRequestAttributes)aRequestContextHolder.getRequestAttributes()).getRequest();
```

- Spring에는 매개변수로 HttpServletRequest를 받지않아도 현재 요청 스레드에서 HttpServletRequest를 가져올수 있는 유틸클래스가 존재한다.
- 해당 유틸클래스에는 메서드가 두개 존재하는데
- getRequestAttributes() 와 currentRequestAttributes() 이다.
- 모두 현재 요청 스레드에서 Request 객체를 가져오는것은 동일하나 getRequestAttributes는 없을시 null을 리턴하고
- currentRequestAttributes 는 예외를 발생시킨다는 차이점이 존재한다.


#### HttpServletRequestUtil
- Spring RequestContextHolder를 활용하여 좀더 쉽고, 유연하게 사용할수 있도록 유틸 Class를 생성해서 사용하면 좋다.

```java
public class RequestScopeUtil {

    public static Attribute getAttribute() {
        return (Attribute) RequestContextHolder.getRequestAttributes().getAttribute(Attribute.KEY, RequestAttributes.SCOPE_REQUEST);
    }

    public static void setAttribute(Attribute attribute) {
        RequestContextHolder.getRequestAttributes().setAttribute(Attribute.KEY, attribute, RequestAttributes.SCOPE_REQUEST);
    }

    public static void removeAttribute() {
        RequestContextHolder.getRequestAttributes().removeAttribute(Attribute.KEY, RequestAttributes.SCOPE_REQUEST);
    }
}
```

#### 마무리
- Spring 에서는 RequestContextHolder Class로 Layer에 상관없이 HttpServletRequest에 접근할수 있도록 기능을 제공한다.
- 개인적인 생각으론 Service, Repository Layer에 HttpServletRequest가 사용되어야한다는것 부터 잘못된 구현이 아닌가 생각이든다. 
- 불가피하게 사용해야한다면 유용할듯 .. 
