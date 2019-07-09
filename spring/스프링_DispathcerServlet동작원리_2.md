# DispatcherServlet 동작원리 2
- 이번에는 ResponseBody가 아닌 view를 리턴하는 핸들러의 경우를 살펴본다

- HellController.class
  - RestController -> Controller 로 변경
  - 기존의 /hello 를 핸들링하던 핸들러에 ResponseBody 애노테이션을 사용하여 기존과 동일하게 요청본문으로 응답하도록 설정
  - /sample 요청을 핸들링하는 핸들러를 작성
  - /WEB-INF/sample.jsp 를 리턴
```java
package me.june;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-08
 * Time: 20:40
 **/
@Controller
public class HelloController {

    @Autowired
    private SampleService sampleService;

    @GetMapping("/hello")
    @ResponseBody
    public String hello () {
        return "Hello " + sampleService.getName();
    }

    @GetMapping("/sample")
    public String sample () {
        return "/WEB-INF/sample.jsp";
    }
}
```

- 이전에 살펴본 /hello 요청을 수행하는 핸들러의 경우에는 modelAndView가 null 이었지만, 이번에는 modelAndView가 null이 아니다.
- @ResponseBody가 없는경우에는 return하는 문자열을 view의 네임으로 인식하고, modelAndView가 핸들러에서 보낸 model을 바인딩해서 view로 보낸다.

- 애노테이션 기반의 핸들러를 등록할수 있는이유는 아무런 설정을 하지않아도, DispatcherServlet이 기본 HandlerMapping을 등록해주기 때문이다.


- SimpleController.class
  - BeanNameHanlderMapping 이 찾는 Handler 구현
  - org.springframework.web.servlet.mvc.Controller 를 구현해야한다.
  - @org.springframework.stereotype.Controller 애노테이션을 활용해서 mapping URL 을 등록한다.
  - SimpleControllerHanlderAdapter 를 사용한다
    - org.springframework.web.servlet.mvc.Controller 를 구현한 핸들러를 처리할수있는 Adapter
```java
package me.june;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-09
 * Time: 21:49
 **/
@org.springframework.stereotype.Controller("/simple")
public class SimpleController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        return new ModelAndView("/WEB-INF/simple.jsp");
    }
}
```

### 정리
- 요청이 들어오면 요청을 분석한다.
- 해당 요청을 수행할수 있는 핸들러를 찾는다.
- @ResponseBody를 사용한 핸들러의 경우 리턴값을 바로 응답본문으로 사용하기때문에 ModelAndView가 null이된다.
- @ResponseBody를 사용하지 '않은' 핸들러의 경우 리턴값을 뷰의 이름으로 인식하고, 해당하는 뷰를찾아서 Model을 바인딩하여 응답한다.
