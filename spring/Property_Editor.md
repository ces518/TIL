# Spring - Property Editor
- Spring MVC 3.0 이전에는 DataBinding을 위해 JavaBeans의 기술인 PropertyEditor 인터페이스를 사용하였다.

- PropertyEditor
    - PropertyEditor를 사용하려면 반드시 PropetyEditorSupport class를 상속받아야하며
    - getAsText() , setAsText() 메서드를 구현해야한다.
    - getAsText() 메서드는 object를 String으로 serializing할때 호출된다.
    - setAsText() 메서드는 String을 object로 변환할때 호출된다.
    - 일시적으로 상태값을 가지며, Thread-safe 하지않다.

- @InitBinder 애노테이션을 사용하여 반드시 classLevel에 설정해 주어야하며, 스프링 빈으로 등록해서 사용해선 안된다.

```java
@InitBinder
public void personPropertyEditor (WebDataBinder webDataBinder) {
    webDataBinder.registerCustomEditor(Person.class, new PersonPropertyEditor());
}
```
