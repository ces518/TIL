# REST API - Event API TEST Class
- TEST Class 생성
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class EventControllerTest {

    @Autowired
    MockMvc mockMvc;
}
```

#### @RunWith
- 테스트 코드는 Junit기반으로 진행
- @RunWith(SpringRunner.class) SpringBoot Test를 위한 SpringRunner class를 지정
```java
package org.junit.runner;

import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * When a class is annotated with <code>&#064;RunWith</code> or extends a class annotated
 * with <code>&#064;RunWith</code>, JUnit will invoke the class it references to run the
 * tests in that class instead of the runner built into JUnit. We added this feature late
 * in development. While it seems powerful we expect the runner API to change as we learn
 * how people really use it. Some of the classes that are currently internal will likely
 * be refined and become public.
 *
 * For example, suites in JUnit 4 are built using RunWith, and a custom runner named Suite:
 *
 * <pre>
 * &#064;RunWith(Suite.class)
 * &#064;SuiteClasses({ATest.class, BTest.class, CTest.class})
 * public class ABCSuite {
 * }
 * </pre>
 *
 * @since 4.0
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Inherited
public @interface RunWith {
    /**
     * @return a Runner class (must have a constructor that takes a single Class to run)
     */
    Class<? extends Runner> value();
}
```

#### @WebMvcTest 
- Web과 관련있는 빈들만 등록이 된다.
- MockMvc가 자동적으로 설정되어 MockMvc를 주입받아 사용할 수 있다.

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot.test.autoconfigure.web.servlet;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.boot.autoconfigure.ImportAutoConfiguration;
import org.springframework.boot.test.autoconfigure.OverrideAutoConfiguration;
import org.springframework.boot.test.autoconfigure.core.AutoConfigureCache;
import org.springframework.boot.test.autoconfigure.filter.TypeExcludeFilters;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.core.annotation.AliasFor;
import org.springframework.test.context.BootstrapWith;
import org.springframework.test.context.junit.jupiter.SpringExtension;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@BootstrapWith(WebMvcTestContextBootstrapper.class)
@ExtendWith({SpringExtension.class})
@OverrideAutoConfiguration(
    enabled = false
)
@TypeExcludeFilters({WebMvcTypeExcludeFilter.class})
@AutoConfigureCache
@AutoConfigureWebMvc
@AutoConfigureMockMvc
@ImportAutoConfiguration
public @interface WebMvcTest {
    String[] properties() default {};

    @AliasFor("controllers")
    Class<?>[] value() default {};

    @AliasFor("value")
    Class<?>[] controllers() default {};

    boolean useDefaultFilters() default true;

    Filter[] includeFilters() default {};

    Filter[] excludeFilters() default {};

    /** @deprecated */
    @Deprecated
    @AliasFor(
        annotation = AutoConfigureMockMvc.class
    )
    boolean secure() default true;

    @AliasFor(
        annotation = ImportAutoConfiguration.class,
        attribute = "exclude"
    )
    Class<?>[] excludeAutoConfiguration() default {};
}
```

#### MockMvc
- Spring Mvc 핵심 클래스
- Mocking되어있는 DispatcherServlet을 상대로 가짜 요청을 만들어 테스트가 가능하다.
- 웹 서버를 띄우지 않기때문에 좀 더 빠르다.
- Slicing Test라고 부른다.

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.test.web.servlet;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import javax.servlet.AsyncContext;
import javax.servlet.DispatcherType;
import javax.servlet.Filter;
import javax.servlet.ServletContext;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpServletResponseWrapper;
import org.springframework.beans.Mergeable;
import org.springframework.lang.Nullable;
import org.springframework.mock.web.MockFilterChain;
import org.springframework.mock.web.MockHttpServletRequest;
import org.springframework.mock.web.MockHttpServletResponse;
import org.springframework.util.Assert;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.DispatcherServlet;

public final class MockMvc {
    static final String MVC_RESULT_ATTRIBUTE = MockMvc.class.getName().concat(".MVC_RESULT_ATTRIBUTE");
    private final TestDispatcherServlet servlet;
    private final Filter[] filters;
    private final ServletContext servletContext;
    @Nullable
    private RequestBuilder defaultRequestBuilder;
    private List<ResultMatcher> defaultResultMatchers = new ArrayList();
    private List<ResultHandler> defaultResultHandlers = new ArrayList();

    MockMvc(TestDispatcherServlet servlet, Filter... filters) {
        Assert.notNull(servlet, "DispatcherServlet is required");
        Assert.notNull(filters, "Filters cannot be null");
        Assert.noNullElements(filters, "Filters cannot contain null values");
        this.servlet = servlet;
        this.filters = filters;
        this.servletContext = servlet.getServletContext();
    }

    void setDefaultRequest(@Nullable RequestBuilder requestBuilder) {
        this.defaultRequestBuilder = requestBuilder;
    }

    void setGlobalResultMatchers(List<ResultMatcher> resultMatchers) {
        Assert.notNull(resultMatchers, "ResultMatcher List is required");
        this.defaultResultMatchers = resultMatchers;
    }

    void setGlobalResultHandlers(List<ResultHandler> resultHandlers) {
        Assert.notNull(resultHandlers, "ResultHandler List is required");
        this.defaultResultHandlers = resultHandlers;
    }

    public DispatcherServlet getDispatcherServlet() {
        return this.servlet;
    }

    public ResultActions perform(RequestBuilder requestBuilder) throws Exception {
        if (this.defaultRequestBuilder != null && requestBuilder instanceof Mergeable) {
            requestBuilder = (RequestBuilder)((Mergeable)requestBuilder).merge(this.defaultRequestBuilder);
        }

        MockHttpServletRequest request = requestBuilder.buildRequest(this.servletContext);
        AsyncContext asyncContext = request.getAsyncContext();
        MockHttpServletResponse mockResponse;
        Object servletResponse;
        if (asyncContext != null) {
            servletResponse = (HttpServletResponse)asyncContext.getResponse();
            mockResponse = this.unwrapResponseIfNecessary((ServletResponse)servletResponse);
        } else {
            mockResponse = new MockHttpServletResponse();
            servletResponse = mockResponse;
        }

        if (requestBuilder instanceof SmartRequestBuilder) {
            request = ((SmartRequestBuilder)requestBuilder).postProcessRequest(request);
        }

        final MvcResult mvcResult = new DefaultMvcResult(request, mockResponse);
        request.setAttribute(MVC_RESULT_ATTRIBUTE, mvcResult);
        RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
        RequestContextHolder.setRequestAttributes(new ServletRequestAttributes(request, (HttpServletResponse)servletResponse));
        MockFilterChain filterChain = new MockFilterChain(this.servlet, this.filters);
        filterChain.doFilter(request, (ServletResponse)servletResponse);
        if (DispatcherType.ASYNC.equals(request.getDispatcherType()) && asyncContext != null && !request.isAsyncStarted()) {
            asyncContext.complete();
        }

        this.applyDefaultResultActions(mvcResult);
        RequestContextHolder.setRequestAttributes(previousAttributes);
        return new ResultActions() {
            public ResultActions andExpect(ResultMatcher matcher) throws Exception {
                matcher.match(mvcResult);
                return this;
            }

            public ResultActions andDo(ResultHandler handler) throws Exception {
                handler.handle(mvcResult);
                return this;
            }

            public MvcResult andReturn() {
                return mvcResult;
            }
        };
    }

    private MockHttpServletResponse unwrapResponseIfNecessary(ServletResponse servletResponse) {
        while(servletResponse instanceof HttpServletResponseWrapper) {
            servletResponse = ((HttpServletResponseWrapper)servletResponse).getResponse();
        }

        Assert.isInstanceOf(MockHttpServletResponse.class, servletResponse);
        return (MockHttpServletResponse)servletResponse;
    }

    private void applyDefaultResultActions(MvcResult mvcResult) throws Exception {
        Iterator var2 = this.defaultResultMatchers.iterator();

        while(var2.hasNext()) {
            ResultMatcher matcher = (ResultMatcher)var2.next();
            matcher.match(mvcResult);
        }

        var2 = this.defaultResultHandlers.iterator();

        while(var2.hasNext()) {
            ResultHandler handler = (ResultHandler)var2.next();
            handler.handle(mvcResult);
        }

    }
}
```

#### 테스트 해야할것 ?
- 입력값을 전달한다면 JSON 응답 201이 나오는가 ?
    - Location 헤더에 생성된 이벤트를 조회할수 있는 URI 가 존재하는가 ?
    - ID는 DB INSERT시 자동생성된 값으로 나오는가 ?
- 입력 값으로 누가 ID, eventStatus, offline, free 와 같은 데이터까지 같이 준다면 ?
    - badRequest or 받기로한 값 이외는 무시
- 입력 데이터가 이상한경우 bad_Request
    - 입력한 값이 이상한가 ?
    - 비즈니스 로직으로 검사가능한 에러 ?
    - 에러 응답 메시지에 에러에 대한 정보가 존재해야한다.
- 비즈니스 로직 적용 되었는지 응답 메시지 확인
    - offline, free 값 확인
- 응답에 HATEOAS와 profile관련 링크가 존재하는가 ?
    - self(view)
    - update(생성한사람은 수정이 가능하다)
    - events(목록으로 이동하는 링크)
- API 문서화
    - 요청 문서화
    - 응답 문서화
    - 링크 문서화
    - profile 링크 추가

#### 테스트 코드
    - mockMvc.perform() // 가짜 요청을 만든다.
    - get(), post(), put() .. 각 HTTP Method에 맞는 가짜 요청을 보낸다.
    - contentType(): 요청 본문의 컨텐츠 타입
    - accept(): Accpet-Header 원하는 응답의 컨텐츠 타입
    - andExcept(): 결과를 assertion 할수 있다.
        - status(): 응답코드
        - isCreated(): 201 응답인지 assertion 한다
        - is(STATUS_CODE): 로 원하는 응답코드를 직접 사용할수 있다.
        - 응답코드를 직접 사용하는것 보다는 각 응답에 대한 메서드들을 사용할것. TYPE-SAFE한 코드가 된다.
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class EventControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void 이벤트생성_테스트 () throws Exception {
        this.mockMvc.perform(post("/api/events/")
                        .contentType(MediaType.APPLICATION_JSON_UTF8) // 요청 본문은 JSON
                        .accept(MediaTypes.HAL_JSON_UTF8) // HAL_JSON 응답을 원한다. (HAL 스펙에 준수하는 응답을 원함)
                    ) 
                    .andExpect(status().isCreated()); // 201 응답
    }
}
```
