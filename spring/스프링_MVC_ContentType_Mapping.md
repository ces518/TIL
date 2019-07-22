# Spring - ContentsType Mapping
- 특정한 타입의 데이터를 담고있는 요청만 처리하도록 매핑
    - @RequestMapping의 consumes 속성을 활용

- consumes에는 문자열이 오는데 문자열을 사용할경우 오탈자가 발생하는등 개발자에 의한 오류가 발생할수 있기때문에 org.springframework.http.MediaType에 정의된 상수를 사용할것을 권장
    - string은 _VALUE로 끝나는 상수를 사용

- org.springframework.http.MediaType.java
```java
/**
 * A subclass of {@link MimeType} that adds support for quality parameters
 * as defined in the HTTP specification.
 *
 * @author Arjen Poutsma
 * @author Juergen Hoeller
 * @author Rossen Stoyanchev
 * @author Sebastien Deleuze
 * @author Kazuki Shimizu
 * @since 3.0
 * @see <a href="https://tools.ietf.org/html/rfc7231#section-3.1.1.1">
 *     HTTP 1.1: Semantics and Content, section 3.1.1.1</a>
 */
public class MediaType extends MimeType implements Serializable {

	private static final long serialVersionUID = 2069937152339670231L;

	/**
	 * Public constant media type that includes all media ranges (i.e. "&#42;/&#42;").
	 */
	public static final MediaType ALL;

	/**
	 * A String equivalent of {@link MediaType#ALL}.
	 */
	public static final String ALL_VALUE = "*/*";

	/**
	 *  Public constant media type for {@code application/atom+xml}.
	 */
	public static final MediaType APPLICATION_ATOM_XML;

	/**
	 * A String equivalent of {@link MediaType#APPLICATION_ATOM_XML}.
	 */
	public static final String APPLICATION_ATOM_XML_VALUE = "application/atom+xml";

	/**
	 * Public constant media type for {@code application/x-www-form-urlencoded}.
	 */
	public static final MediaType APPLICATION_FORM_URLENCODED;

	/**
	 * A String equivalent of {@link MediaType#APPLICATION_FORM_URLENCODED}.
	 */
	public static final String APPLICATION_FORM_URLENCODED_VALUE = "application/x-www-form-urlencoded";

	/**
	 * Public constant media type for {@code application/json}.
	 * @see #APPLICATION_JSON_UTF8
	 */
	public static final MediaType APPLICATION_JSON;

	/**
	 * A String equivalent of {@link MediaType#APPLICATION_JSON}.
	 * @see #APPLICATION_JSON_UTF8_VALUE
	 */
	public static final String APPLICATION_JSON_VALUE = "application/json";

	/**
	 * Public constant media type for {@code application/json;charset=UTF-8}.
	 *
	 * <p>This {@link MediaType#APPLICATION_JSON} variant should be used to set JSON
	 * content type because while
	 * <a href="https://tools.ietf.org/html/rfc7159#section-11">RFC7159</a>
	 * clearly states that "no charset parameter is defined for this registration", some
	 * browsers require it for interpreting correctly UTF-8 special characters.
	 */
	public static final MediaType APPLICATION_JSON_UTF8;
    ...
    ...
```

- ContentType
    - HttpHeader중 ContentType 은 해당 본문의 DataType을 알려주는 역할을 한다.

- JSON 요청만 처리하도록 하는 Handler
```java
@Controller
public class MvcController {

    @GetMapping(value = "/hello", consumes = MediaType.APPLICATION_JSON_UTF8_VALUE)
    @ResponseBody
    public String hello () {
        return "hello";
    }
}
```

- Test
```java
@Test
public void helloTest () throws Exception {
    this.mockMvc.perform(get("/hello"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(content().string("hello"));
}
```

- 결과
    - 415 UnSupportedMediaType 응답
    - 해당 컨텐츠타입을 지원하지않음. 
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /hello
       Parameters = {}
          Headers = []
             Body = <no character encoding set>
    Session Attrs = {}

Handler:
             Type = null

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = org.springframework.web.HttpMediaTypeNotSupportedException

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 415
    Error message = null
          Headers = [Accept:"application/json;charset=UTF-8"]
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

- 일반적으로 web에서 요청을보낼경우 text/html 로 요청을 보내게됨.
- ContentType 을 application/json 으로 테스트진행

```java
@Test
public void helloTest () throws Exception {
    this.mockMvc.perform(get("/hello")
                .contentType(MediaType.APPLICATION_JSON))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(content().string("hello"));
}
```


- 결과
    - 200 응답
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /hello
       Parameters = {}
          Headers = [Content-Type:"application/json"]
             Body = <no character encoding set>
    Session Attrs = {}

Handler:
             Type = me.june.springbootmvc.mvc.MvcController
           Method = public java.lang.String me.june.springbootmvc.mvc.MvcController.hello()

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"text/plain;charset=UTF-8", Content-Length:"5"]
     Content type = text/plain;charset=UTF-8
             Body = hello
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

- 특정한 타입의 응답을 만드는 핸들러
    - @RequestMapping(produces="application/json")
    - accpet Header로 필터링

- TEXT/PLANE 의 응답만 처리하는 핸들러 코드 작성
```java
@Controller
public class MvcController {

    @GetMapping(
            value = "/hello",
            consumes = MediaType.APPLICATION_JSON_UTF8_VALUE,
            produces = MediaType.TEXT_PLAIN_VALUE)
    @ResponseBody
    public String hello () {
        return "hello";
    }
}
```

- 테스트
    - application/json 응답을 받도록 요청
```java
@Test
public void helloTest () throws Exception {
    this.mockMvc.perform(get("/hello")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_JSON_UTF8))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(content().string("hello"));
}
```

- 결과
    - 406 NotSupported 응답
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /hello
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/json;charset=UTF-8"]
             Body = null
    Session Attrs = {}

Handler:
             Type = null

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = org.springframework.web.HttpMediaTypeNotAcceptableException

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 406
    Error message = null
          Headers = []
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

#### 애매한점
- accept Header가 정의되지 않은경우, 모든 응답을 받는다는 의미이기때문에 에러가 발생하지 않는다.

```java
@Test
public void helloTest () throws Exception {
    this.mockMvc.perform(get("/hello")
                .contentType(MediaType.APPLICATION_JSON_UTF8))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(content().string("hello"));
}
```

- 결과
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /hello
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8"]
             Body = null
    Session Attrs = {}

Handler:
             Type = me.june.springbootmvc.mvc.MvcController
           Method = public java.lang.String me.june.springbootmvc.mvc.MvcController.hello()

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"text/plain;charset=UTF-8", Content-Length:"5"]
     Content type = text/plain;charset=UTF-8
             Body = hello
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```


- consumes는 ClassLevel에 선언한 @RequestMapping에 사용한 것과 조합이 되지않고, MethodLevel에 선언한 @RequestMapping의 설정으로 덮어쓴다.
