# REST API - Spring - HATEOAS 적용하기
- REST가 잘적용된 API 라면 응답에 HATEOAS를 지켜야한다. Spring - HATEOAS 를 사용하여 HATEOAS 적용하자

#### 테스트 코드 변경
- 링크 정보를 제공하는지 테스트 코드를 추가한다.
    - self: 리소스 에 대한 링크
    - query-events: 이벤트목록에 대한 링크
    - update-event: 이벤트 수정에 대한 링크
- 현재는 아무런 링크정보도 제공하지 않기때문에 당연히 테스트는 실패한다.

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

    // EventRepository를 Mocking했기때문에 return값을 mocking해주어야함
//        given(eventRepository.save(any(Event.class))).willReturn(event);

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
    ;
}
```

#### EventResource
- EventResource를 쉽게 생성하는 방법은 ResourceSupport를 상속받는 클래스를 새롭게 정의하고, Event 객체를 주입받아 Getter 메서드를 활용하여 제공하는 방법.
- ResourceSupport를 상속받으면 add Method를 통해 링크정보를 추가할 수 있다.
    - withRel(): 이 링크가 리소스와 어떤 관계에 있는지 관계를 정의할 수 있다.
    - withSelfRel(): 리소스에 대한 링크를 type-safe 한 method로 제공한다.
- 링크 정보에는 어떠한 Method를 사용해야하는지에 대한 정보는 제공할 수 없다.
- Relation과 HREF 만 제공할 수 있음.
```java
@Getter
public class EventResource extends ResourceSupport {

    private Event event;

    public EventResource(Event event) {
        this.event = event;
    }
}

@PostMapping
public ResponseEntity createEvent (@Valid @RequestBody EventDto eventDto, Errors errors) { // 입력값을 EventDto를 활용하여 받는다.
    eventValidator.validate(eventDto, errors);
    if (errors.hasErrors()) {
        return ResponseEntity.badRequest().body(errors);
    }

    Event event = objectMapper.convertValue(eventDto, Event.class);
    // 비즈니스 로직을 수행
    event.update();

    Event savedEvent = eventRepository.save(event);
    // created를 생성할때는 항상 uri를 제공해야한다.
    // org.springframework.hateoas.mvc.ControllerLinkBuilder 를 사용하면 손쉽게 URI를 생성할 수 있음.
    ControllerLinkBuilder linkBuilder = linkTo(EventController.class).slash(savedEvent.getId()); // 새로 생성된 Event의 ID를 기반으로 Location Header로 들어간다.
    URI uri = linkBuilder.toUri();

    /* 링크정보에는 어떤 Method를 사용해야하는지에 대한 정보는 담을 수 없다. */
    EventResource eventResource = new EventResource(savedEvent);
    eventResource.add(linkTo(EventController.class).withRel("query-events"));
    eventResource.add(linkBuilder.withSelfRel());
    eventResource.add(linkBuilder.withRel("update-event"));
    return ResponseEntity.created(uri).body(eventResource);
}
```

#### 테스트 결과
- 테스트가 실패한다.
- 응답을 보면, event에 대한 정보들과, 링크정보들이 존재한다. 
- 하지만 왜 실패할까 ?
    - 응답을 보낼때 jackson (ObjectMapper) 를 사용하여 Serialization을 진행한다.
    - 즉 BeanSerializer를 사용하는데 BeanSerializer는 기본적으로 필드명을 사용한다. 따라서 Test Assertion조건에 맞지않다.
        - 응답 내부에 event가 존재하고 event에 정보가 있는 구조이다.
```java
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events/
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/hal+json;charset=UTF-8"]
             Body = {"id":100,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":false,"free":false,"eventStatus":"PUBLISHED"}
    Session Attrs = {}

Handler:
             Type = me.june.restapi.events.EventController
           Method = public org.springframework.http.ResponseEntity me.june.restapi.events.EventController.createEvent(me.june.restapi.events.EventDto,org.springframework.validation.Errors)

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
           Status = 201
    Error message = null
          Headers = [Location:"http://localhost/api/events/1", Content-Type:"application/hal+json;charset=UTF-8"]
     Content type = application/hal+json;charset=UTF-8
             Body = {"event":{"id":1,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":true,"free":false,"eventStatus":"DRAFT"},"_links":{"query-events":{"href":"http://localhost/api/events"},"self":{"href":"http://localhost/api/events/1"},"update-event":{"href":"http://localhost/api/events/1"}}}
    Forwarded URL = null
   Redirected URL = http://localhost/api/events/1
          Cookies = []

MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events/
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/hal+json;charset=UTF-8"]
             Body = {"id":100,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":false,"free":false,"eventStatus":"PUBLISHED"}
    Session Attrs = {}

Handler:
             Type = me.june.restapi.events.EventController
           Method = public org.springframework.http.ResponseEntity me.june.restapi.events.EventController.createEvent(me.june.restapi.events.EventDto,org.springframework.validation.Errors)

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
           Status = 201
    Error message = null
          Headers = [Location:"http://localhost/api/events/1", Content-Type:"application/hal+json;charset=UTF-8"]
     Content type = application/hal+json;charset=UTF-8
             Body = {"event":{"id":1,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":true,"free":false,"eventStatus":"DRAFT"},"_links":{"query-events":{"href":"http://localhost/api/events"},"self":{"href":"http://localhost/api/events/1"},"update-event":{"href":"http://localhost/api/events/1"}}}
    Forwarded URL = null
   Redirected URL = http://localhost/api/events/1
          Cookies = []



java.lang.AssertionError: No value at JSON path "$.id"
```

#### 해결방법
- 1. Event의 필드들을 그대로 EventResource에 옮겨담는 방법
    - 매우 단순하고 1차원적인 해결방법
    - 나쁜 방법은 아니다.
- 2. @JsonUnwrapped 애노테이션 사용
    - fieldName을 사용하지않고 wrap 되지 상태로 serialize 된다.
- 3. Resource<T>
    - ResourceSupport 하위 클래스에 Resource<T> 라는 클래스가 존재한다.
    - T에 해당하는 데이터가 content 로 매핑이 되는데 getContent() Method에 @JsonUnwrapped 가 붙어있기때문에 unwrap이 된다.

- @JsonUnwrapped
```java
package com.fasterxml.jackson.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.ANNOTATION_TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotation
public @interface JsonUnwrapped
{
    /**
     * Property that is usually only used when overriding (masking) annotations,
     * using mix-in annotations. Otherwise default value of 'true' is fine, and
     * value need not be explicitly included.
     */
    boolean enabled() default true;

    /**
     * Optional property that can be used to add prefix String to use in front
     * of names of properties that are unwrapped: this can be done for example to prevent
     * name collisions.
     */
    String prefix() default "";

    /**
     * Optional property that can be used to add suffix String to append at the end
     * of names of properties that are unwrapped: this can be done for example to prevent
     * name collisions.
     */
    String suffix() default "";
}
```
```java
@Getter
public class EventResource extends ResourceSupport {

    @JsonUnwrapped
    private Event event;

   public EventResource(Event event) {
       this.event = event;
   }
}
```

- Resource<T>
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.hateoas;

import com.fasterxml.jackson.annotation.JsonUnwrapped;
import java.util.Arrays;
import java.util.Collection;
import javax.xml.bind.annotation.XmlAnyElement;
import javax.xml.bind.annotation.XmlRootElement;
import org.springframework.util.Assert;

@XmlRootElement
public class Resource<T> extends ResourceSupport {
    private final T content;

    Resource() {
        this.content = null;
    }

    public Resource(T content, Link... links) {
        this(content, (Iterable)Arrays.asList(links));
    }

    public Resource(T content, Iterable<Link> links) {
        Assert.notNull(content, "Content must not be null!");
        Assert.isTrue(!(content instanceof Collection), "Content must not be a collection! Use Resources instead!");
        this.content = content;
        this.add(links);
    }

    @JsonUnwrapped
    @XmlAnyElement
    public T getContent() {
        return this.content;
    }

    public String toString() {
        return String.format("Resource { content: %s, %s }", this.getContent(), super.toString());
    }

    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        } else if (obj != null && obj.getClass().equals(this.getClass())) {
            Resource<?> that = (Resource)obj;
            boolean contentEqual = this.content == null ? that.content == null : this.content.equals(that.content);
            return contentEqual ? super.equals(obj) : false;
        } else {
            return false;
        }
    }

    public int hashCode() {
        int result = super.hashCode();
        result += this.content == null ? 0 : 17 * this.content.hashCode();
        return result;
    }
}
```
```java
@Getter
public class EventResource extends Resource<Event> {

    private Event event;

    public EventResource(Event content, Link... links) {
        super(content, links);
    }

//    public EventResource(Event event) {
//        this.event = event;
//    }
}
```

#### 테스트 결과
- 더 이상 wrap 되지않은 구조로 응답이 오기때문에 테스트는 성공한다.
- 응답 헤더를 보면 hal+json 이기 때문에 클라이언트가 이 리소스는 링크정보를 제공한다고 인지할수 있기때문에 링크정보를 활용할 수 있음.
```java
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events/
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/hal+json;charset=UTF-8"]
             Body = {"id":100,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":false,"free":false,"eventStatus":"PUBLISHED"}
    Session Attrs = {}

Handler:
             Type = me.june.restapi.events.EventController
           Method = public org.springframework.http.ResponseEntity me.june.restapi.events.EventController.createEvent(me.june.restapi.events.EventDto,org.springframework.validation.Errors)

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
           Status = 201
    Error message = null
          Headers = [Location:"http://localhost/api/events/1", Content-Type:"application/hal+json;charset=UTF-8"]
     Content type = application/hal+json;charset=UTF-8
             Body = {"id":1,"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-08-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-08-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100,"offline":true,"free":false,"eventStatus":"DRAFT","_links":{"query-events":{"href":"http://localhost/api/events"},"self":{"href":"http://localhost/api/events/1"},"update-event":{"href":"http://localhost/api/events/1"}}}
    Forwarded URL = null
   Redirected URL = http://localhost/api/events/1
          Cookies = []
```

#### SpringBoot의 HATEOAS 자동 설정
- @EnableHypermediaSupport.. 등의 애노테이션을 사용하여 설정을 해주어야만 사용할수 있지만 SpringBoot가 자동설정을 해주기때문에 추가적인 설정 없이도 HATEOAS를 사용할 수 있다.
