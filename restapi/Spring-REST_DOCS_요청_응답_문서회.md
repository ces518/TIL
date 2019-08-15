# REST API - Spring - REST DOCS_요청_문서화
#### REST Docs 자동설정
    - @AutoConfigureRestDocs
    - Spring Boot를 사용한다면 별다른 설정 없이 @AutoConfigureRestDocs 애노테이션만 사용하면 Rest Docs를 사용할 수 있다.

- @AutoConfigureRestDocs
```java
package org.springframework.boot.test.autoconfigure.restdocs;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.boot.autoconfigure.ImportAutoConfiguration;
import org.springframework.boot.test.autoconfigure.properties.PropertyMapping;
import org.springframework.context.annotation.Import;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@ImportAutoConfiguration
@Import({RestDocumentationContextProviderRegistrar.class})
@PropertyMapping("spring.test.restdocs")
public @interface AutoConfigureRestDocs {
    @AliasFor("outputDir")
    String value() default "";

    @AliasFor("value")
    String outputDir() default "";

    String uriScheme() default "http";

    String uriHost() default "localhost";

    int uriPort() default 8080;
}
```

```java
@RunWith(SpringRunner.class)
//@WebMvcTest
@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureRestDocs
public class EventControllerTest {
    ...
}
```

#### 간단한 Snippets 생성 코드
- .andDo(document("create-event")): create-event 라는 이름의 snippets를 생성하도록 코드를 수정한다.
- 테스트를 실행하면 create-evnet snippets가 생성된다.
```java
    @Test
    @TestDescription("정상적인 이벤트 생성 테스트")
    public void 이벤트생성_테스트 () throws Exception {
        Event event = Event.builder()
                .name("Spring")
                .description("REST API Study")
                .beginEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
                .closeEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
                .beginEventDateTime(LocalDateTime.of(2019, 8, 15, 14, 21))
                .endEventDateTime(LocalDateTime.of(2019, 8, 16, 14, 21))
                .basePrice(100)
                .maxPrice(200)
                .limitOfEnrollment(100)
                .location("대전 둔산동 스타벅스")
                .eventStatus(EventStatus.PUBLISHED)
                .build();

        event.setId(100);

        String eventJsonString = objectMapper.writeValueAsString(event);

        this.mockMvc.perform(post("/api/events/")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .accept(MediaTypes.HAL_JSON_UTF8)
                        .content(eventJsonString)
                    )
                    .andDo(print())
                    .andExpect(status().isCreated()) // 201 응답
                    .andExpect(header().exists(HttpHeaders.LOCATION))
                    .andExpect(header().string(HttpHeaders.CONTENT_TYPE, MediaTypes.HAL_JSON_UTF8_VALUE))
                    .andExpect(jsonPath("$.id").value(Matchers.not(100))) // 입력값이 들어와선 안된다.
                    .andExpect(jsonPath("$.free").value(false)) // 유료 이벤트
                    .andExpect(jsonPath("$.offline").value(true)) // 오프라인
                    .andExpect(jsonPath("$.eventStatus").value(EventStatus.DRAFT.name()))
                    .andExpect(jsonPath("$._links.self").exists())
                    .andExpect(jsonPath("$._links.query-events").exists())
                    .andExpect(jsonPath("$._links.update-event").exists())
                    .andDo(document("create-event"))
        ;
    }
```

#### Snippets 생성 위치
- Spring - Rest Docs를 사용하여 테스트 코드를 실행하면 기본적으로 target/generated-snippets 디렉터리 하위에 생성된다.

#### 테스트 결과
- RestDocs를 적용하고 테스트코드를 실행하면 snippets하위에 다음과 같은 파일이 생성된다.
- curl-request.adoc
    - 리눅스 curl 
- http-request.adoc
    - http-request
- http-response.adoc
    - http-response
- httpie-request.adoc
    - 리눅스 httpie
- request-body.adoc
    - request-body
- response-body.adoc
    - response-body


- http-request.adoc
```java
[source,http,options="nowrap"]
----
POST /api/events/ HTTP/1.1
Content-Length: 390
Content-Type: application/json;charset=UTF-8
Accept: application/hal+json;charset=UTF-8
Host: localhost:8080

{"id":100,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":false,"free":false,"eventStatus":"PUBLISHED"}
----
```

#### 문제점
- 현재 생성된 snippets만 보더라도 Client입장에서는 API 요청과 응답에 대한 정보를 알수 있지만, Formatting이 되어 있지않아 보기가 불편하다.
- Formatting을 하려면 RestDocksMockMvc를 커스터마이징 해주어야한다.

#### RestDocMockMvc 커스터마이징
- RestDocsMockMvcConfigurationCustomizer 를 구현하는 빈을 등록해야한다.
- @TestConfiguration 을 사용하여 테스트용 설정 클래스를 작성
    - @TestConfiguration 클래스는 test/ 하위 패키지에 존재해야한다.
- prettyPrint() 설정으로 요청과 본문을 보기 편하게 설정을 해준다.

```java
@TestConfiguration
public class RestDocsConfiguration {

    @Bean
    public RestDocsMockMvcConfigurationCustomizer restDocsMockMvcConfigurationCustomizer () {
        return new RestDocsMockMvcConfigurationCustomizer() {
            @Override
            public void customize(MockMvcRestDocumentationConfigurer configurer) {
                configurer.operationPreprocessors()
                        .withRequestDefaults(prettyPrint())
                        .withResponseDefaults(prettyPrint());
            }
        };
    }
}
```

#### Formatting 결과
- @Import 를 사용하여 다른 Bean 설정을 읽어주어야 해당 설정이 적용이 된다.
```java
@RunWith(SpringRunner.class)
//@WebMvcTest
@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureRestDocs
@Import(RestDocsConfiguration.class)
public class EventControllerTest {

}
```

- 설정이 적용된 후 결과
    - 다음과 같이 보기 편하게 Formatting이 된것을 확인할 수 있다.
```java
[source,http,options="nowrap"]
----
HTTP/1.1 201 Created
Location: http://localhost:8080/api/events/1
Content-Length: 709
Content-Type: application/hal+json;charset=UTF-8

{
  "id" : 1,
  "name" : "Spring",
  "description" : "REST API Study",
  "beginEnrollmentDateTime" : "2019-08-05T11:23:00",
  "closeEnrollmentDateTime" : "2019-08-05T11:23:00",
  "beginEventDateTime" : "2019-08-15T14:21:00",
  "endEventDateTime" : "2019-08-16T14:21:00",
  "location" : "대전 둔산동 스타벅스",
  "basePrice" : 100,
  "maxPrice" : 200,
  "limitOfEnrollment" : 100,
  "offline" : true,
  "free" : false,
  "eventStatus" : "DRAFT",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/api/events/1"
    },
    "query-events" : {
      "href" : "http://localhost:8080/api/events"
    },
    "update-event" : {
      "href" : "http://localhost:8080/api/events/1"
    }
  }
}
----
```

- 이 외에도 Preprocessors 들이 존재한다.
- MaskingLinks.. 등
