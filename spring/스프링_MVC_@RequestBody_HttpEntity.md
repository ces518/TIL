# Spring - @RequestBody & HttpEntity
- 요청 본문에 있는 내용을 HttpMessageConverter를 사용해서 변환후 객체로 받아올 수 있다.

- @RequestBody
	- 요청본문에 존재하는 데이터를 HttpMessageConverter를 통해 변환한 객체로 받아올 수 있다.
	- @Valid, @Valiated로 유효성 검사 가능
	- BindingResult 를 사용해 바인딩 또는 검증 에러에 대한 핸들링이 가능하다.
	- @RequestBody를 사용시 HttpMessageConverter를 사용하여 Conversion을 시도한다.

```java
@PostMapping("event")
public String create (@RequestBody Event event) {
	...
}
```

- HttpMessageConverter
	- SpringMVC 설정 (WebMvcConfigurer) 에서 설정
	- configureMessageConverters(): 기본 메시지 컨버터 대체
	- extendMessageConverters(): 기본 메시지 컨버터 유지 후 추가 설정
	- 기본 컨버터
		- WebMvcConfigurationSupport.addDefaultHttpMessageConverters
	- HandlerAdapter가 사용
		- Handler Method Argument를 Resolve 할때 (HandlerMethodArgumentResolver) HttpMessageConverter를 사용한다.
		- 현재 요청에 알맞는 Converter를 사용해서 변환을 시도한다.

- Jackson2HttpMessageConverter
	- JSON 요청 과 응답시 사용하는 MessageConverter
	- Spring boot 를 사용하면 Jackson2ObjectMapper가 classpath에 의존성으로 존재한다.
		- ObjectMapper가 Bean으로 등록된다.
	- Jackson2HttpMessageConverter가 자동적으로 등록된다.

```java
@Configuration
class JacksonHttpMessageConvertersConfiguration {

	@Configuration
	@ConditionalOnClass(ObjectMapper.class)
	@ConditionalOnBean(ObjectMapper.class)
	@ConditionalOnProperty(name = HttpMessageConvertersAutoConfiguration.PREFERRED_MAPPER_PROPERTY,
			havingValue = "jackson", matchIfMissing = true)
	protected static class MappingJackson2HttpMessageConverterConfiguration {

		@Bean
		@ConditionalOnMissingBean(value = MappingJackson2HttpMessageConverter.class,
				ignoredType = { "org.springframework.hateoas.mvc.TypeConstrainedMappingJackson2HttpMessageConverter",
						"org.springframework.data.rest.webmvc.alps.AlpsJsonHttpMessageConverter" })
		public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter(ObjectMapper objectMapper) {
			return new MappingJackson2HttpMessageConverter(objectMapper);
		}

	}

	@Configuration
	@ConditionalOnClass(XmlMapper.class)
	@ConditionalOnBean(Jackson2ObjectMapperBuilder.class)
	protected static class MappingJackson2XmlHttpMessageConverterConfiguration {

		@Bean
		@ConditionalOnMissingBean
		public MappingJackson2XmlHttpMessageConverter mappingJackson2XmlHttpMessageConverter(
				Jackson2ObjectMapperBuilder builder) {
			return new MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build());
		}

	}

}
```

- Handler Method Argument Resolver
	- parameter를 Binding하는 역할을 한다.

- HttpEntity
	- Generic Type에 해당하는 본문으로 받아올수 있음
	- @RequestBody와 유사하지만, 추가적으로 요청 헤더 정보를 사용할 수 있다.
```java
@PostMapping("event")
public String create (HttpEntity<Event> request) {
	request.getHeaders().getContentType();
}
```
