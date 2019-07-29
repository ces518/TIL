# Spring Handler Method - MultipartFile
- MultipartFile
    - FileUpload 처리시 사용할 수 있는 Handler Method ArgumentType이다.
    - 이를 사용하려면 DispatcherServlet에 MulitpartResolver가 빈으로 등록 되어 있어야 한다.
    - DispatcherServlet의 initMultipartResolver 부분에서 설정을 한다.
    - 기본 전략은 MultipartResolver = null

- initMultipartResolver
```java
private void initMultipartResolver(ApplicationContext context) {
    try {
        this.multipartResolver = (MultipartResolver)context.getBean("multipartResolver", MultipartResolver.class);
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Detected " + this.multipartResolver);
        } else if (this.logger.isDebugEnabled()) {
            this.logger.debug("Detected " + this.multipartResolver.getClass().getSimpleName());
        }
    } catch (NoSuchBeanDefinitionException var3) {
        this.multipartResolver = null;
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("No MultipartResolver 'multipartResolver' declared");
        }
    }

}
```

- SpringBoot는 MultipartAutoConfiguration class에 의해 자동으로 설정된다.
```java
/**
 * {@link EnableAutoConfiguration Auto-configuration} for multi-part uploads. Adds a
 * {@link StandardServletMultipartResolver} if none is present, and adds a
 * {@link javax.servlet.MultipartConfigElement multipartConfigElement} if none is
 * otherwise defined. The {@link ServletWebServerApplicationContext} will associate the
 * {@link MultipartConfigElement} bean to any {@link Servlet} beans.
 * <p>
 * The {@link javax.servlet.MultipartConfigElement} is a Servlet API that's used to
 * configure how the server handles file uploads.
 *
 * @author Greg Turnquist
 * @author Josh Long
 * @author Toshiaki Maki
 */
@Configuration
@ConditionalOnClass({ Servlet.class, StandardServletMultipartResolver.class, MultipartConfigElement.class })
@ConditionalOnProperty(prefix = "spring.servlet.multipart", name = "enabled", matchIfMissing = true)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(MultipartProperties.class)
public class MultipartAutoConfiguration {

	private final MultipartProperties multipartProperties;

	public MultipartAutoConfiguration(MultipartProperties multipartProperties) {
		this.multipartProperties = multipartProperties;
	}

	@Bean
	@ConditionalOnMissingBean({ MultipartConfigElement.class, CommonsMultipartResolver.class })
	public MultipartConfigElement multipartConfigElement() {
		return this.multipartProperties.createMultipartConfig();
	}

	@Bean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME)
	@ConditionalOnMissingBean(MultipartResolver.class)
	public StandardServletMultipartResolver multipartResolver() {
		StandardServletMultipartResolver multipartResolver = new StandardServletMultipartResolver();
		multipartResolver.setResolveLazily(this.multipartProperties.isResolveLazily());
		return multipartResolver;
	}

}
```

- MultipartResolver의 상세 Property 들은 MultipartProperties class를 참조하기때문에
- Application.properties 에서 커스터마이징이 가능하다.
```java
@ConfigurationProperties(prefix = "spring.servlet.multipart", ignoreUnknownFields = false)
public class MultipartProperties {

	/**
	 * Whether to enable support of multipart uploads.
	 */
	private boolean enabled = true;

	/**
	 * Intermediate location of uploaded files.
	 */
	private String location;

	/**
	 * Max file size.
	 */
	private DataSize maxFileSize = DataSize.ofMegabytes(1);

	/**
	 * Max request size.
	 */
	private DataSize maxRequestSize = DataSize.ofMegabytes(10);

	/**
	 * Threshold after which files are written to disk.
	 */
	private DataSize fileSizeThreshold = DataSize.ofBytes(0);

	/**
	 * Whether to resolve the multipart request lazily at the time of file or parameter
	 * access.
	 */
	private boolean resolveLazily = false;

	public boolean getEnabled() {
		return this.enabled;
	}

	public void setEnabled(boolean enabled) {
		this.enabled = enabled;
	}

	public String getLocation() {
		return this.location;
	}

	public void setLocation(String location) {
		this.location = location;
	}

	public DataSize getMaxFileSize() {
		return this.maxFileSize;
	}

	public void setMaxFileSize(DataSize maxFileSize) {
		this.maxFileSize = maxFileSize;
	}

	public DataSize getMaxRequestSize() {
		return this.maxRequestSize;
	}

	public void setMaxRequestSize(DataSize maxRequestSize) {
		this.maxRequestSize = maxRequestSize;
	}

	public DataSize getFileSizeThreshold() {
		return this.fileSizeThreshold;
	}

	public void setFileSizeThreshold(DataSize fileSizeThreshold) {
		this.fileSizeThreshold = fileSizeThreshold;
	}

	public boolean isResolveLazily() {
		return this.resolveLazily;
	}

	public void setResolveLazily(boolean resolveLazily) {
		this.resolveLazily = resolveLazily;
	}

	/**
	 * Create a new {@link MultipartConfigElement} using the properties.
	 * @return a new {@link MultipartConfigElement} configured using there properties
	 */
	public MultipartConfigElement createMultipartConfig() {
		MultipartConfigFactory factory = new MultipartConfigFactory();
		PropertyMapper map = PropertyMapper.get().alwaysApplyingWhenNonNull();
		map.from(this.fileSizeThreshold).to(factory::setFileSizeThreshold);
		map.from(this.location).whenHasText().to(factory::setLocation);
		map.from(this.maxRequestSize).to(factory::setMaxRequestSize);
		map.from(this.maxFileSize).to(factory::setMaxFileSize);
		return factory.createMultipartConfig();
	}

}
```

- POST multipart/form-data 요청에 들어있는 파일을 참조할 수 있다.
- List<MultipartFile> 형태로 여러 파일을 참조할 수 도 있다.

- FileUpload 처리 Handler
```java
@PostMapping("/file")
@ResponseBody
public String upload (MultipartFile file) {
    String fileName = file.getOriginalFilename();
    // fileUpload Code ....
    return fileName + "is Uploaded!!";
}
```
