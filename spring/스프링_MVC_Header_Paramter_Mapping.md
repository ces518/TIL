# Spring - Header, Parameter Mapping
- 특정한 헤더와 관련된 요청을 매핑하고싶은경우 @RequestMapping 애노테이션의 headers 속성을 활용해서 매핑을 하면된다.
- headers에 오는 값의 경우 org.springframework.http.HttpHeaders 에 정의된 상수가 존재하기때문에 해당 상수를 활용할것

- org.springframework.http.HttpHeaders
```java
public class HttpHeaders implements MultiValueMap<String, String>, Serializable {

	private static final long serialVersionUID = -8578554704772377436L;


	/**
	 * The HTTP {@code Accept} header field name.
	 * @see <a href="https://tools.ietf.org/html/rfc7231#section-5.3.2">Section 5.3.2 of RFC 7231</a>
	 */
	public static final String ACCEPT = "Accept";
	/**
	 * The HTTP {@code Accept-Charset} header field name.
	 * @see <a href="https://tools.ietf.org/html/rfc7231#section-5.3.3">Section 5.3.3 of RFC 7231</a>
	 */
	public static final String ACCEPT_CHARSET = "Accept-Charset";
	/**
	 * The HTTP {@code Accept-Encoding} header field name.
	 * @see <a href="https://tools.ietf.org/html/rfc7231#section-5.3.4">Section 5.3.4 of RFC 7231</a>
	 */
	public static final String ACCEPT_ENCODING = "Accept-Encoding";
	/**
	 * The HTTP {@code Accept-Language} header field name.
	 * @see <a href="https://tools.ietf.org/html/rfc7231#section-5.3.5">Section 5.3.5 of RFC 7231</a>
	 */
	public static final String ACCEPT_LANGUAGE = "Accept-Language";
	/**
	 * The HTTP {@code Accept-Ranges} header field name.
	 * @see <a href="https://tools.ietf.org/html/rfc7233#section-2.3">Section 5.3.5 of RFC 7233</a>
	 */
	public static final String ACCEPT_RANGES = "Accept-Ranges";
	/**
	 * The CORS {@code Access-Control-Allow-Credentials} response header field name.
	 * @see <a href="https://www.w3.org/TR/cors/">CORS W3C recommendation</a>
	 */
	public static final String ACCESS_CONTROL_ALLOW_CREDENTIALS = "Access-Control-Allow-Credentials";
	/**
	 * The CORS {@code Access-Control-Allow-Headers} response header field name.
	 * @see <a href="https://www.w3.org/TR/cors/">CORS W3C recommendation</a>
	 */
	public static final String ACCESS_CONTROL_ALLOW_HEADERS = "Access-Control-Allow-Headers";
	/**
	 * The CORS {@code Access-Control-Allow-Methods} response header field name.
	 * @see <a href="https://www.w3.org/TR/cors/">CORS W3C recommendation</a>
	 */
	public static final String ACCESS_CONTROL_ALLOW_METHODS = "Access-Control-Allow-Methods";
    ...
```

- AUTHORIZATION 이라는 Header가 존재하는 경우에만 매핑이 되도록 핸들러 코드 작성
```java
@Controller
public class MvcController {

    @GetMapping(value = "/hello", headers = HttpHeaders.AUTHORIZATION)
    @ResponseBody
    public String hello () {
        return "hello";
    }
}
```

- AUTHORIZATION 이라는 Header가 없는 요청을 테스트
```java
    @Test
    public void helloTest () throws Exception {
        this.mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("hello"));
    }
```
- 테스트 결과
    - 404 NotFound 응답
    - 다음과 같이 매핑되는 핸들러가 없다는 응답을 리턴
```java
MockHttpServletResponse:
           Status = 404
    Error message = null
          Headers = []
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /hello
       Parameters = {}
          Headers = []
             Body = <no character encoding set>
    Session Attrs = {}

Handler:
             Type = org.springframework.web.servlet.resource.ResourceHttpRequestHandler

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
           Status = 404
    Error message = null
          Headers = []
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

- 특정한 헤더가 있는 요청 매핑하기
    - @RequestMapping(header = "key")

- 특정한 헤더가 없는 요청 매핑하기
    - @RequestMapping(header = "!key")

- 특정한 헤더 키/값이 있는 요청 매핑하기
    - @RequestMapping(header = "key=value")




- 특정한 파라메터와 관련된 요청을 매핑하고 싶은경우 @RequestMapping 의 params 속성을 활용해서 매핑을 하면된다.


- name이라는 parameter가 존재하는 경우에만 매핑이되도록 핸들러 코드 작성
```java
@Controller
public class MvcController {

    @GetMapping(value = "/hello", params = "name")
    @ResponseBody
    public String hello () {
        return "hello";
    }
}
```


- 아무런 파라메터도 보내지않는 요청을 테스트
```java
@Test
public void helloTest () throws Exception {
    this.mockMvc.perform(get("/hello"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(content().string("hello"));
}
```

- 테스트 결과
    - 400 BAD_REQUEST 응답
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
             Type = org.springframework.web.bind.UnsatisfiedServletRequestParameterException

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 400
    Error message = Parameter conditions "name" not met for actual request parameters: 
          Headers = []
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

- 특정한 요청 매개변수 키를 가지고있는 요청을 매핑하기
    - @RequestMapping(params = "a")

- 특정한 요청 매개변수가 없는 요청을 매핑하기
    - @RequestMapping(params = "!a")

- 특정한 요청 매개변수 키/값이 있는 요청 매핑하기
    - @RequesetMapping(params = "key=value")
