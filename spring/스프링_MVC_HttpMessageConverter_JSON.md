# Spring MVC - HttpMessageConverter - JSON
- HttpMessageConverter는 의존성에 따라 조건적으로 등록이 된다.

- SpringBoot를 사용하는경우
    - 기본적으로 JacksonJSON2 가 의존성에 들어있다.
    - JSON용 HTTP MessageConverter가 기본으로 등록되어있다.


- Handler 작성
    - GET /jsonMessage 으로 요청을 받는다.
    - 해당 요청의 본문을 읽어 HttpMessageConverter를 사용하여 Person객체로 받는다.
    - person객체를 HttpMessageConverter를 사용해서 응답해주는 핸들러
```java
@GetMapping("/jsonMessage")
public Person jsonMessage (@RequestBody Person person) {
    return person;
}
```

- TestCode 작성
    - SpringBoot를 사용하면 Jackson이 의존성에 들어와있기 때문에 ObjectMapper를 주입받아 사용할수 있다.
    - contentType: 요청을 보내는 컨텐츠타입을 명시
    - content: 요청의 본문
    - GET /jsonMessage { "name": "june" } 요청에 대한 테스트코드
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Autowired
    ObjectMapper objectMapper;


    @Test
    public void jsonMessage () throws Exception {

        Person person = new Person();
        person.setName("june");

        String jsonString = objectMapper.writeValueAsString(person);

        this.mockMvc.perform(get("/jsonMessage")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(jsonString))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```

- 결과
```java
MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/json;charset=UTF-8"]
     Content type = application/json;charset=UTF-8
             Body = {"name":"june"}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
