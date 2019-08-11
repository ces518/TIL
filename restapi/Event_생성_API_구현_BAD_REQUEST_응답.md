# REST API - Event 생성 API 구현 - BAD_REQUEST 응답
- BAD_REQUEST 응답을 보냈지만, 응답만 봐서는 어떤 원인에 의해 BAD_REQUEST가 발생했는지에 대한 정보가 제공되고 있지 않다.
- 잘못된 요청이라면 어떤 원인에의해 잘못된 요청인지 응답과 함께 제공해야한다.

#### 테스트 코드
- 잘못된 요청을 보냈을경우 해당 에러 정보가 존재하는지 테스트
- 이러한 에러에 대한 정보는 Errors객체가 가지고있다.
```java
@Test
@TestDescription("이벤트 생성 시작일이 종료일을 넘을경우 테스트")
public void 이벤트생성_시작일이_종료일을_넘을경우_BAD_REQUEST () throws Exception {
    // 입력값을 아무것도 보내지않을 경우 테스트
    EventDto eventDto = EventDto.builder()
            .name("Spring")
            .description("REST API Study")
            .beginEnrollmentDateTime(LocalDateTime.of(2019, 10 , 5, 11, 23))
            .closeEnrollmentDateTime(LocalDateTime.of(2019, 8 , 5, 11, 23))
            .beginEventDateTime(LocalDateTime.of(2019, 10, 15, 14, 21))
            .endEventDateTime(LocalDateTime.of(2019, 8, 16, 14, 21))
            .basePrice(100)
            .maxPrice(200)
            .limitOfEnrollment(100)
            .location("대전 둔산동 스타벅스")
            .build();

    this.mockMvc.perform(post("/api/events")
            .contentType(MediaType.APPLICATION_JSON_UTF8)
            .accept(MediaTypes.HAL_JSON_UTF8)
            .content(objectMapper.writeValueAsString(eventDto)))
            .andDo(print())
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$[0].objectName").exists())
            .andExpect(jsonPath("$[0].field").exists())
            .andExpect(jsonPath("$[0].defaultMessage").exists())
            .andExpect(jsonPath("$[0].code").exists())
            .andExpect(jsonPath("$[0].rejectedValue").exists())
    ;
}
```

#### 이벤트 생성 API 변경
- Error가 발생하면 Errors객체를 응답본문으로 제공하면 되지않나 ?
    - ObjectMapper는 다양한 Serializer를 가지고있는데 Errors객체는 'Java Bean Spec' 을 준수하는 객체가 아니기때문에
    - Serialization이 불가능하다.
```java
@PostMapping
public ResponseEntity createEvent (@Valid @RequestBody EventDto eventDto, Errors errors) { // 입력값을 EventDto를 활용하여 받는다.
    eventValidator.validate(eventDto, errors);
    if (errors.hasErrors()) {
        return ResponseEntity.badRequest().body(errors);
    }

    Event event = objectMapper.convertValue(eventDto, Event.class);

    Event savedEvent = eventRepository.save(event);
    // created를 생성할때는 항상 uri를 제공해야한다.
    // org.springframework.hateoas.mvc.ControllerLinkBuilder 를 사용하면 손쉽게 URI를 생성할 수 있음.
    ControllerLinkBuilder linkBuilder = linkTo(EventController.class).slash(savedEvent.getId()); // 새로 생성된 Event의 ID를 기반으로 Location Header로 들어간다.
    URI uri = linkBuilder.toUri();
    return ResponseEntity.created(uri).body(savedEvent);
}
```


#### ObjectMapper
- ObjectMapper는 JavaBean Spec을 준수하는 객체의 경우 기본으로 등록된 Bean Serializer 를 사용해서 Serialization을 진행한다.


#### 결과
- 적절한 Serializer를 찾지 못했다는 예외가 발생하게 된다.
```java
Caused by: com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.springframework.validation.DefaultMessageCodesResolver and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: org.springframework.validation.BeanPropertyBindingResult["messageCodesResolver"])
	at com.fasterxml.jackson.databind.exc.InvalidDefinitionException.from(InvalidDefinitionException.java:77)
	at com.fasterxml.jackson.databind.SerializerProvider.reportBadDefinition(SerializerProvider.java:1191)
	at com.fasterxml.jackson.databind.DatabindContext.reportBadDefinition(DatabindContext.java:313)
	at com.fasterxml.jackson.databind.ser.impl.UnknownSerializer.failForEmpty(UnknownSerializer.java:71)
	at com.fasterxml.jackson.databind.ser.impl.UnknownSerializer.serialize(UnknownSerializer.java:33)
	at com.fasterxml.jackson.databind.ser.BeanPropertyWriter.serializeAsField(BeanPropertyWriter.java:727)
	at com.fasterxml.jackson.databind.ser.std.BeanSerializerBase.serializeFields(BeanSerializerBase.java:719)
	at com.fasterxml.jackson.databind.ser.BeanSerializer.serialize(BeanSerializer.java:155)
	at com.fasterxml.jackson.databind.ser.DefaultSerializerProvider._serialize(DefaultSerializerProvider.java:480)
	at com.fasterxml.jackson.databind.ser.DefaultSerializerProvider.serializeValue(DefaultSerializerProvider.java:319)
	at com.fasterxml.jackson.databind.ObjectWriter$Prefetch.serialize(ObjectWriter.java:1396)
	at com.fasterxml.jackson.databind.ObjectWriter.writeValue(ObjectWriter.java:913)
	at org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.writeInternal(AbstractJackson2HttpMessageConverter.java:287)
	... 62 more
```


#### Java Bean 이란 ?
- https://github.com/ces518/ILE/blob/master/java/JavaBean.md


#### Errors
- Errors를 활용하여 에러 정보를 담는데는 2가지 유형이 존재한다.
- 1. FieldError
    - rejectValue() Method를 사용하여 에러 정보를 담은경우
    - 제공하는 정보 
        - Field
        - ObjectName
        - Code
        - DefaultMessage
- 2. GlobalError
    - reject() Method를 사용하여 에러 정보를 담은경우
    - 제공하는 정보
        - ObjectName
        - Code
        - DefaultMessage
```java
public void validate (EventDto eventDto, Errors errors) {
    // 무제한 경매가 아닌데, basePrice 가 max보다 큰 경우
    if (eventDto.getBasePrice() > eventDto.getMaxPrice() && eventDto.getMaxPrice() > 0) {
        // fieldError
        errors.rejectValue("basePrice", "wrongValue", "BasePrice is must be less than MaxPrice");
        // globalError
        errors.reject("basePrice Error !!");
    }

    LocalDateTime endEventDateTime = eventDto.getEndEventDateTime();
    if (endEventDateTime.isBefore(eventDto.getBeginEventDateTime()) ||
    endEventDateTime.isBefore(eventDto.getCloseEnrollmentDateTime()) ||
    endEventDateTime.isBefore(eventDto.getBeginEnrollmentDateTime())) {
        errors.rejectValue("endEventDateTime", "wrongValue", "EndEventDateTime is must be more than begin");
    }

}
```

#### Errors를 Serialization 하는 방법
- 1. 별도의 ErrorResponse 객체를 만들어 Error 관련 정보를 내보내는 방법
    - 일종의 DTO를 사용하는방법
- 2. * Custom Serializer를 만드는 방법
    - ObjectMapper가 사용한 Serializer를 만들어 등록하는 방법


#### Custom Serializer
- JsonSerializer<T> 를 구현하는 클래스를 생성후 serialize() Method를 구현한다.
```java
@JsonComponent
public class ErrorsSerializer extends JsonSerializer<Errors> {

    @Override
    public void serialize(Errors errors, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        // 배열로 존재하기때문에 startArray, endArray를 사용
        jsonGenerator.writeStartArray();

        // Field Errors
        errors.getFieldErrors().stream().forEach(e -> {
                try {
                    jsonGenerator.writeStartObject();
                    jsonGenerator.writeStringField("field", e.getField());
                    jsonGenerator.writeStringField("objectName", e.getObjectName());
                    jsonGenerator.writeStringField("code", e.getCode());
                    jsonGenerator.writeStringField("defaultMessage", e.getDefaultMessage());
                    Object rejectedValue = e.getRejectedValue();
                    if (rejectedValue != null) {
                        jsonGenerator.writeStringField("rejectedValue", rejectedValue.toString());
                    }
                    jsonGenerator.writeEndObject();
                } catch (IOException e1) {
                    e1.printStackTrace();
                }
        });

    // GlobalErrors
        errors.getGlobalErrors().stream().forEach(e -> {
            try {
                jsonGenerator.writeStartObject();
                jsonGenerator.writeStringField("objectName", e.getObjectName());
                jsonGenerator.writeStringField("code", e.getCode());
                jsonGenerator.writeStringField("defaultMessage", e.getDefaultMessage());
                jsonGenerator.writeEndObject();
            } catch (IOException e1) {
                e1.printStackTrace();
            }
        });

        jsonGenerator.writeEndArray();
    }
}
```

#### @JsonComponent
- ObjectMapper 에 Custom Serializer를 등록해 주어야하는데 Spring Boot에서 제공하는 @JsonComponent를 사용하면 손쉽게 등록이 가능하다.
- https://www.baeldung.com/spring-boot-jsoncomponent


#### Custom Serializer 를 적용한 후 테스트 결과
- EventValidator 에서 Validation 한 결과 (에러 정보)가 응답 본문에 함께 제공되는것을 볼 수 있다.
```java
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /api/events
       Parameters = {}
          Headers = [Content-Type:"application/json;charset=UTF-8", Accept:"application/hal+json;charset=UTF-8"]
             Body = {"name":"Spring","description":"REST API Study","beginEnrollmentDateTime":"2019-10-05T11:23:00","closeEnrollmentDateTime":"2019-08-05T11:23:00","beginEventDateTime":"2019-10-15T14:21:00","endEventDateTime":"2019-08-16T14:21:00","location":"대전 둔산동 스타벅스","basePrice":100,"maxPrice":200,"limitOfEnrollment":100}
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
           Status = 400
    Error message = null
          Headers = [Content-Type:"application/hal+json;charset=UTF-8"]
     Content type = application/hal+json;charset=UTF-8
             Body = [{"field":"endEventDateTime","objectName":"eventDto","code":"wrongValue","defaultMessage":"EndEventDateTime is must be more than begin","rejectedValue":"2019-08-16T14:21"}]
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
