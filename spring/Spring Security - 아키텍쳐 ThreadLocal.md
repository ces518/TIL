# Spring Security - ArcheTecher_ThreadLocal
- java.lang 패키지에서 제공하는 Thread Scope Variable 
- 쓰레드 수준의 데이터 저장소이다.
- SecurityContextHolder의 기본 전략이다.

같은 쓰레드 내에서만 공유
따라서 같은 쓰레드라면 해당 데이터를 메서드 매개변수로 사용하지않아도 데이터를 전달할 수 있다.

#### Context 생성
- ThreadLocal을 사용해서 한 쓰레드내에서 Account객체를 공유 할 수 있는 Context를 구현한다.
- Account 객체를 ThreadLocal에 저장하고, 가져오는 로직만 존재하는 매우 간단한 클래스이다.
```java
public class AccountContext {
    private static final ThreadLocal<Account> ACCOUNT_THREAD_LOCAL = new ThreadLocal<>();

    public static void setAccount (Account account) {
        ACCOUNT_THREAD_LOCAL.set(account);
    }

    public static Account getAccount () {
        return ACCOUNT_THREAD_LOCAL.get();
    }
}
```

#### Context 활용
- SampleService의 dashboard() 메서드 수정하기

dashboard() 메서드를 호출하면 AccountContext에 저장된 account 객체를 꺼내와 username을 출력하는 매우 간단한 메서드이다.
```java
@Service
public class SampleService {

    public void dashboard () {
        /*
            AccountContext에 저장된 account객체를 꺼내온다.
        */
        Account account = AccountContext.getAccount();
        System.out.println("username = " + account.getUsername());
    }
}
```

- SampleController 의 dashboard api를 수정해서 dashboard() 메서드를 활용하기

수정된 SampleController는 다음과 같다. dashboard API를 자세히 살펴보자
```java
@Controller
@RequiredArgsConstructor
public class SampleController {

    private final AccountRepository accountRepository;
    private final SampleService sampleService;

    @GetMapping("/")
    public String index (Model model, Principal principal) {
        if (principal == null) {
            model.addAttribute("message", "Hello Spring Security");
        } else {
            model.addAttribute("message", "Hello Index" + principal.getName());
        }
        return "index";
    }

    @GetMapping("/info")
    public String info (Model model) {
        model.addAttribute("message", "Hello Info");
        return "info";
    }

    @GetMapping("/dashboard")
    public String dashboard (Model model, Principal principal) {
        model.addAttribute("message", "Hello " + principal.getName());
        /*
          AccountRepository에서 조회한 Account객체를 AccountContext에 저장한다.
        */
        AccountContext.setAccount(accountRepository.findByUsername(principal.getName()));
        sampleService.dashboard();
        return "dashboard";
    }

    @GetMapping("/admin")
    public String admin (Model model, Principal principal) {
        model.addAttribute("message", "Hello Admin" + principal.getName());
        return "admin";
    }
}
```

- /dashboard로 요청을 보내면 accountRepository에서 account를 새롭게 조회한다.
- 조회한 Account객체를 AccountContext에 저장한다.
- sampleService의 dashboard() 메서드를 호출한다.
```java
@GetMapping("/dashboard")
public String dashboard (Model model, Principal principal) {
	model.addAttribute("message", "Hello " + principal.getName());
	/*
		AccountRepository에서 조회한 Account객체를 AccountContext에 저장한다.
	*/
	AccountContext.setAccount(accountRepository.findByUsername(principal.getName()));
	sampleService.dashboard();
	return "dashboard";
}
```

#### 출력 결과
```java
2019-09-13 23:09:10.151  INFO 1233 --- [  restartedMain] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2019-09-13 23:09:10.183  INFO 1233 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2019-09-13 23:09:10.631  WARN 1233 --- [  restartedMain] aWebConfiguration$JpaWebMvcConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2019-09-13 23:09:10.667  INFO 1233 --- [  restartedMain] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
2019-09-13 23:09:10.715  INFO 1233 --- [  restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: any request, [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@4c0b5e0b, org.springframework.security.web.context.SecurityContextPersistenceFilter@180d3c25, org.springframework.security.web.header.HeaderWriterFilter@5cdd1104, org.springframework.security.web.csrf.CsrfFilter@4e2d7221, org.springframework.security.web.authentication.logout.LogoutFilter@15a1009d, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@5d2876a3, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@1b43cbf, org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@6ec7336f, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@226a1ac3, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@206adb0f, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@fc8320f, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@3663ffcf, org.springframework.security.web.session.SessionManagementFilter@71cc76dc, org.springframework.security.web.access.ExceptionTranslationFilter@2ac75c4d, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@5270cd98]
2019-09-13 23:09:10.816  INFO 1233 --- [  restartedMain] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-09-13 23:09:11.030  INFO 1233 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-09-13 23:09:11.034  INFO 1233 --- [  restartedMain] m.j.s.SpringSecurityApplication          : Started SpringSecurityApplication in 3.463 seconds (JVM running for 9.174)
2019-09-13 23:11:14.418  INFO 1233 --- [nio-8080-exec-2] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2019-09-13 23:11:14.419  INFO 1233 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2019-09-13 23:11:14.426  INFO 1233 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Completed initialization in 7 ms
2019-09-13 23:11:54.861  INFO 1233 --- [nio-8080-exec-8] o.h.h.i.QueryTranslatorFactoryInitiator  : HHH000397: Using ASTQueryTranslatorFactory
username = user
```


ThreadLocal을 활용하여 직접 AccountContext를 구현해 보았다.
스프링 시큐리티에서는 SecurityContextHolder가 SecurityContext를 활용하여 Authentication 객체를 알아서 저장해주기 때문에 SecurityContextHolder를 이용해 Authentication 객체에 접근이 가능한 것이다.
