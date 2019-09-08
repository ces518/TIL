# Spring Security - InMemory User 사용하기
- 시큐리티가 기본적으로 생성해주는 User를 사용하는것이 아닌 우리가 직접 생성한 유저정보를 사용하도록 설정한다.


#### 기본 시큐리티 유저 커스터마이징 하기
- 스프링 부트 애플리케이션을 실행하면 다음과 같이 Log에 매번 새로운 비밀번호가 설정된 것이 남는다.
- 이러한 자동 설정은 어디서 오는것일까 ? 
- 로그에서 그 힌트를 제공하고 있다.
    - UserDetailsServiceAutoConfiguration 
```java
2019-09-08 14:36:50.999  INFO 826 --- [  restartedMain] .s.s.UserDetailsServiceAutoConfiguration : 

Using generated security password: 825a69ad-c708-4d76-82ae-3b03f5d24463
```

#### UserDetailsServiceAutoConfiguration
- 다음은 우리가 Log에서 힌트를 얻었던 UserDetailsServiceAutoConfiguration 클래스이다.
- 이중에서 유심히 살펴봐야 할것은 inMemoryUserDetailsManager Method 이다.
- 좀더 자세히 살펴보자.
```java
/**
 * {@link EnableAutoConfiguration Auto-configuration} for a Spring Security in-memory
 * {@link AuthenticationManager}. Adds an {@link InMemoryUserDetailsManager} with a
 * default user and generated password. This can be disabled by providing a bean of type
 * {@link AuthenticationManager}, {@link AuthenticationProvider} or
 * {@link UserDetailsService}.
 *
 * @author Dave Syer
 * @author Rob Winch
 * @author Madhura Bhave
 * @since 2.0.0
 */
@Configuration
@ConditionalOnClass(AuthenticationManager.class)
@ConditionalOnBean(ObjectPostProcessor.class)
@ConditionalOnMissingBean({ AuthenticationManager.class, AuthenticationProvider.class, UserDetailsService.class })
public class UserDetailsServiceAutoConfiguration {

	private static final String NOOP_PASSWORD_PREFIX = "{noop}";

	private static final Pattern PASSWORD_ALGORITHM_PATTERN = Pattern.compile("^\\{.+}.*$");

	private static final Log logger = LogFactory.getLog(UserDetailsServiceAutoConfiguration.class);

	@Bean
	@ConditionalOnMissingBean(
			type = "org.springframework.security.oauth2.client.registration.ClientRegistrationRepository")
	@Lazy
	public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties,
			ObjectProvider<PasswordEncoder> passwordEncoder) {
		SecurityProperties.User user = properties.getUser();
		List<String> roles = user.getRoles();
		return new InMemoryUserDetailsManager(
				User.withUsername(user.getName()).password(getOrDeducePassword(user, passwordEncoder.getIfAvailable()))
						.roles(StringUtils.toStringArray(roles)).build());
	}

	private String getOrDeducePassword(SecurityProperties.User user, PasswordEncoder encoder) {
		String password = user.getPassword();
		if (user.isPasswordGenerated()) {
			logger.info(String.format("%n%nUsing generated security password: %s%n", user.getPassword()));
		}
		if (encoder != null || PASSWORD_ALGORITHM_PATTERN.matcher(password).matches()) {
			return password;
		}
		return NOOP_PASSWORD_PREFIX + password;
	}

}
```

- SecurityProperties로 부터 User 정보를 받아 기본 유저 정보를 생성하는것 처럼 보인다.
- 그렇다면 SecurityProperties 클래스는 어떻게 구성되어 있을까 ?
```java
@Bean
@ConditionalOnMissingBean(
        type = "org.springframework.security.oauth2.client.registration.ClientRegistrationRepository")
@Lazy
public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties,
        ObjectProvider<PasswordEncoder> passwordEncoder) {
    SecurityProperties.User user = properties.getUser();
    List<String> roles = user.getRoles();
    return new InMemoryUserDetailsManager(
            User.withUsername(user.getName()).password(getOrDeducePassword(user, passwordEncoder.getIfAvailable()))
                    .roles(StringUtils.toStringArray(roles)).build());
}
```

#### SecurityProperties
- spring.security 를 prefix로 설정을 주입받아 사용 할 수 있다.
```java
@ConfigurationProperties(prefix = "spring.security")
public class SecurityProperties {

	/**
	 * Order applied to the WebSecurityConfigurerAdapter that is used to configure basic
	 * authentication for application endpoints. If you want to add your own
	 * authentication for all or some of those endpoints the best thing to do is to add
	 * your own WebSecurityConfigurerAdapter with lower order.
	 */
	public static final int BASIC_AUTH_ORDER = Ordered.LOWEST_PRECEDENCE - 5;

	/**
	 * Order applied to the WebSecurityConfigurer that ignores standard static resource
	 * paths.
	 */
	public static final int IGNORED_ORDER = Ordered.HIGHEST_PRECEDENCE;

	/**
	 * Default order of Spring Security's Filter in the servlet container (i.e. amongst
	 * other filters registered with the container). There is no connection between this
	 * and the {@code @Order} on a WebSecurityConfigurer.
	 */
	public static final int DEFAULT_FILTER_ORDER = OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER - 100;

	private final Filter filter = new Filter();

	private User user = new User();

	public User getUser() {
		return this.user;
	}

	public Filter getFilter() {
		return this.filter;
	}

	public static class Filter {

		/**
		 * Security filter chain order.
		 */
		private int order = DEFAULT_FILTER_ORDER;

		/**
		 * Security filter chain dispatcher types.
		 */
		private Set<DispatcherType> dispatcherTypes = new HashSet<>(
				Arrays.asList(DispatcherType.ASYNC, DispatcherType.ERROR, DispatcherType.REQUEST));

		public int getOrder() {
			return this.order;
		}

		public void setOrder(int order) {
			this.order = order;
		}

		public Set<DispatcherType> getDispatcherTypes() {
			return this.dispatcherTypes;
		}

		public void setDispatcherTypes(Set<DispatcherType> dispatcherTypes) {
			this.dispatcherTypes = dispatcherTypes;
		}

	}

	public static class User {

		/**
		 * Default user name.
		 */
		private String name = "user";

		/**
		 * Password for the default user name.
		 */
		private String password = UUID.randomUUID().toString();

		/**
		 * Granted roles for the default user name.
		 */
		private List<String> roles = new ArrayList<>();

		private boolean passwordGenerated = true;

		public String getName() {
			return this.name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public String getPassword() {
			return this.password;
		}

		public void setPassword(String password) {
			if (!StringUtils.hasLength(password)) {
				return;
			}
			this.passwordGenerated = false;
			this.password = password;
		}

		public List<String> getRoles() {
			return this.roles;
		}

		public void setRoles(List<String> roles) {
			this.roles = new ArrayList<>(roles);
		}

		public boolean isPasswordGenerated() {
			return this.passwordGenerated;
		}

	}
}
```

예상 한것과 마찬가지로 내부적으로 랜덤하게 생성하고 있었다.
- 패스워드를 지정해준다면 passwordGenerated 가 false가 되며 더이상 유저가 생성되지 않는다.
- 그렇다면 정말 그런 설정이 가능한지 한번 테스트를 진행해보자.
```java
public static class User {

    /**
        * Default user name.
        */
    private String name = "user";

    /**
        * Password for the default user name.
        */
    private String password = UUID.randomUUID().toString();

    /**
        * Granted roles for the default user name.
        */
    private List<String> roles = new ArrayList<>();

    private boolean passwordGenerated = true;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return this.password;
    }

    public void setPassword(String password) {
        if (!StringUtils.hasLength(password)) {
            return;
        }
        this.passwordGenerated = false;
        this.password = password;
    }

    public List<String> getRoles() {
        return this.roles;
    }

    public void setRoles(List<String> roles) {
        this.roles = new ArrayList<>(roles);
    }

    public boolean isPasswordGenerated() {
        return this.passwordGenerated;
    }

}
```

다음과 같이 application.properties 파일에 설정을 진행한다.

```properties
spring.security.user.name=admin
spring.security.user.password=1234
spring.security.user.roles=ADMIN
```

설정을 진행하고 난뒤 다시 스프링부트 애플리케이션을 실행하면, 이전과는 달리 패스워드가 자동설정되던 Log가 없다.
```java
2019-09-08 14:44:53.403  INFO 880 --- [  restartedMain] m.j.s.SpringSecurityApplication          : Starting SpringSecurityApplication on bagjun-yeong-ui-MacBook-Pro.local with PID 880 (/Users/june/spring-security-form/target/classes started by june in /Users/june/spring-security-form)
2019-09-08 14:44:53.405  INFO 880 --- [  restartedMain] m.j.s.SpringSecurityApplication          : No active profile set, falling back to default profiles: default
2019-09-08 14:44:53.435  INFO 880 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2019-09-08 14:44:53.435  INFO 880 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2019-09-08 14:44:54.239  INFO 880 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-09-08 14:44:54.261  INFO 880 --- [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-09-08 14:44:54.262  INFO 880 --- [  restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.22]
2019-09-08 14:44:54.321  INFO 880 --- [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-09-08 14:44:54.321  INFO 880 --- [  restartedMain] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 886 ms
2019-09-08 14:44:54.555  INFO 880 --- [  restartedMain] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
2019-09-08 14:44:54.590  INFO 880 --- [  restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: any request, [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@362e9563, org.springframework.security.web.context.SecurityContextPersistenceFilter@2b55dd30, org.springframework.security.web.header.HeaderWriterFilter@25bee3aa, org.springframework.security.web.csrf.CsrfFilter@fa2ed4a, org.springframework.security.web.authentication.logout.LogoutFilter@596b734c, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@24949336, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@54077e1e, org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@5a43083b, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@2af6c3ba, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@20958f7b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@267b903f, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@51a2ea28, org.springframework.security.web.session.SessionManagementFilter@1ece0907, org.springframework.security.web.access.ExceptionTranslationFilter@5a9a6670, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@5463a9c8]
2019-09-08 14:44:54.668  INFO 880 --- [  restartedMain] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-09-08 14:44:54.779  INFO 880 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2019-09-08 14:44:54.839  INFO 880 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-09-08 14:44:54.843  INFO 880 --- [  restartedMain] m.j.s.SpringSecurityApplication          : Started SpringSecurityApplication in 1.647 seconds (JVM running for 7.166)
```

#### 결과
- http://localhost:8080/admin 으로 요청을 보내면 폼 인증을 위한 시큐리티 기본 로그인폼이 나타난다.
- admin/1234 로 인증을 진행하면 인증이 완료되고 ADMIN role을 가지고 있기때문에 admin 페이지로 접근이 가능해진다.
- 하지만 유저를 1명이상 생성이 불가능하다.

#### 유저 를 여러명 생성하기
- properties 에 설정된 값들을 모두 제거하고 SecurityConfig 클래스로 이동한다.
- AuthenticationManagerBuilder 를 가지는 configure Method를 Override 한다.
- 인증 방법은 다양한다.
    - inMemory
    - Jdbc
    - Ldap 
    - ...
- 그중에서 inMemory를 사용한다.
- 이전 properties 설정과 크게 다르진 않지만 눈에 띄는 설정은 바로 password 부분이다.
- password 부분을 좀 더 살펴보자.
```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
            .withUser("admin").password("{noop}1234").roles("ADMIN").and()
            .withUser("user").password("{noop}user1234").roles("USER");
}
```

- password 앞에 prefix로 {noop} 이라는 문자열이 붙는다.
- 왜 그럴까 ?
    - Security 5.x 부터 기본 패스워드 인코더가 변경되었다.
    - 이전과는 방식이 바뀌어 해당 패스워드가 어떠한 암호화방식으로 암호화 되었는지를 알려주는 prefix 문자열이다.
    - 해당 prefix문자열이 없다면, 잘못된 값이라고 판단, 예외를 발생시킨다.
```java
.withUser("admin").password("{noop}1234").roles("ADMIN")
```


#### 해결되지 않은 문제
- 유저가 추가될때마다 설정파일을 열어 추가해야한다.
