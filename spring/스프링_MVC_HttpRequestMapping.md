# Spring - HttpRequestMapping
- Http요청을 처리하는 역할을 하는것을 handler라고 한다.

- 가장 간단한 핸들러 코드작성
    - @RequestMapping 애노테이션을 활용하여 /hello 요청에 대한 핸들러 코드작성
    - @RequestMapping 사용시 HttpMethod를 설정하지않을경우 기본적으로 모든 method에 대한 요청을 받는다.

```java
@Controller
public class MvcController {

    @RequestMapping("/hello")
    @ResponseBody
    public String hello () {
        return "hello";
    }
}
```

- 간단한 테스트코드 작성
    - @RunWith(SpringRunner.class)
        - SpringBoot Test를 실행할수있도록 도와주는 JUnitClass 테스트용 ApplicationContext를 생성해준다.
    - @WebMvcTest
        - Web과 관련된 테스트 애노테이션, Web과 관련된 Bean들만 등록이된다.
        - MockMvc객체를 주입받을수 있다.
    - MockMvc를 통해서 MockHttpServletRequest를 보내 slicingTest가 가능하다.
    - GET /hello 로 요청을 보내는 테스트
    - print(): 요청과 응답의 결과 출력
    - andExpect: 기대 결과값
        - status().isOk(): 상태값이 200이기를 기대
        - content().string("hello"): 응답 본문에 hello 문자열이 존재하는것을 기대
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class MvcControllerTest {
    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest () throws Exception {
        this.mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("hello"));
    }
}
```

- 테스트 결과
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /hello
       Parameters = {}
          Headers = []
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

- 특정 Method만 허용하고싶은경우 RequestMapping에 method속성을 활용한다.
    - org.springframework.web.bind.annotation의 RequestMethod enum을 활용한다.

- RequestMethod.java

```java
package org.springframework.web.bind.annotation;

/**
 * Java 5 enumeration of HTTP request methods. Intended for use with the
 * {@link RequestMapping#method()} attribute of the {@link RequestMapping} annotation.
 *
 * <p>Note that, by default, {@link org.springframework.web.servlet.DispatcherServlet}
 * supports GET, HEAD, POST, PUT, PATCH and DELETE only. DispatcherServlet will
 * process TRACE and OPTIONS with the default HttpServlet behavior unless explicitly
 * told to dispatch those request types as well: Check out the "dispatchOptionsRequest"
 * and "dispatchTraceRequest" properties, switching them to "true" if necessary.
 *
 * @author Juergen Hoeller
 * @since 2.5
 * @see RequestMapping
 * @see org.springframework.web.servlet.DispatcherServlet#setDispatchOptionsRequest
 * @see org.springframework.web.servlet.DispatcherServlet#setDispatchTraceRequest
 */
public enum RequestMethod {

	GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE

}
```

- GET /hello 로 요청을 제한하도록 핸들러코드 수정 

```java
@Controller
public class MvcController {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    @ResponseBody
    public String hello () {
        return "hello";
    }
}
```

- PUT /hello로 테스트진행
```java
    @Test
    public void helloTest () throws Exception {
        this.mockMvc.perform(put("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("hello"));
    }
```

- 결과
    - 405 MethodNotAllowed 응답
    - Header [Allow:"GET"] 허용된 메서드정보도 함께 응답한다.
```java
MockHttpServletRequest:
      HTTP Method = PUT
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
             Type = org.springframework.web.HttpRequestMethodNotSupportedException

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 405
    Error message = Request method 'PUT' not supported
          Headers = [Allow:"GET"]
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

- 다수의 Method를 허용하고싶을경우 배열을 활용하여 선언한다.
```java
@RequestMapping(value = "/hello", method = {RequestMethod.GET, RequestMethod.POST})
@ResponseBody
public String hello () {
    return "hello";
}
```

- 하나의 Method만 허용하고 싶은경우 Meta애노테이션을 활용하여 좀더 간략하게 사용가능하다.
    - GetMapping
    - PostMapping
    - ...

- @RequestMapping을 ClassLevel에 설정이 가능하다.
    - 해당 class에 존재하는 모든 핸들러의 baseURL이 된다.
    - ClassLevel에 Method를 제한하는경우 해당 클래스의 모든 Handler는 해당 Method로 요청이 제한된다.


#### HttpMethod

- GET
    - 클라이언트가 서버의 리소스를 요청할때 사용함
    - 캐싱이 가능하다. 캐시와 관련된 헤더를 응답에 실어보낼수 있음.
    - 브라우저 기록에 남는다.
    - 북마크가 가능하다.
    - 민감한 데이터를 보낼때 사용하지말것.
    - idemponent (동일 요청에 대한 응답이 같음)

- POST
    - 클라이언트가 서버의 리소르를 수정하거나, 새로 생성할때 사용함
    - 서버에 보내는 데이터를 POST 요청 본문에 담는다.
    - 캐싱이 불가능하다.
    - 브라우저 기록에 남지않는다.
    - 북마크 할 수 없다.
    - 데이터 길이 제한이없다.
    
- PUT
    - URI에 해당하는 데이터를 새로 만들거나, 수정할 때 사용한다.
    - POST와 다른점은 URI에 대한 의미가 다르다.
        - POST의 URI는 보내는 데이터를 처리할 리소스를 지칭
        - PUT의 URI는 보내는 데이터에 해당하는 리소스를 지칭
    - idemponent

- PATCH
    - PUT과 비슷하지만, 기조 ㄴ엔티티와 새 데이터의 차이점만 보낸다.
        - 부분수정
    - idemponent

- DELETE
    - URI에 해당하는 리소스를 삭제할 때 사용한다.
    - idemponent
