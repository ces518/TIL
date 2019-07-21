# Spring MVC - HttpMessageConverter - XML
- OXM (Object - XML - Mapper) 라이브러리중 스프링이 지원하는 의존성 추가
    - jacksonXML
    - JAXB

- JAXB 의존성 추가
```xml
<!-- jaxb interface -->
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
</dependency>
<!-- 구현체 -->
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
</dependency>
<!-- 마샬링, 추상화된 spring-oxm -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-oxm</artifactId>
    <version>${spring-framework.version}</version>
</dependency>
```

- Mashaller를 빈으로 등록해서 해당 빈을 사용할것

- Jaxb2Marshaller 를 빈으로 등록
    - setPackagesToScan jaxb를 사용할 패키지를 설정해주어야한다.
    - @XmlRootElement로 알려주어야지 jaxb가 변환이 가능하다.
```java
@Bean
public Jaxb2Marshaller jaxb2Marshaller () {
    Jaxb2Marshaller jaxb2Marshaller = new Jaxb2Marshaller();
    jaxb2Marshaller.setPackagesToScan(Person.class.getPackage().getName());
    return jaxb2Marshaller;
}

@XmlRootElement
@Getter @Setter
public class Person {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

- TestCode
    - org.springframework.oxm.Marshaller 를 빈으로 주입받는다.
        - 이전에 등록한 Jaxb2Marshaller이 주입된다.
    - Marshaller를 사용하려면 marshal 메서드를 사용해야한다.
        - 첫번째 인자로 마샬링 대상 Object, 두번째로 해당 결과를 받을 Result 객체를 받는다.
        - stringWriter로 받은 결과를 toString 을 통해 문자열로 변환이 가능하다.
```java
    @Autowired
    Marshaller marshaller;

    @Test
    public void xmlMessage () throws Exception {
        Person person = new Person();
        person.setName("june");

        StringWriter stringWriter = new StringWriter();
        Result result = new StreamResult(stringWriter);
        marshaller.marshal(person, result);

        String xmlString = stringWriter.toString();

        this.mockMvc.perform(get("/jsonMessage")
                .contentType(MediaType.APPLICATION_XML)
                .accept(MediaType.APPLICATION_XML)
                .content(xmlString))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(xpath("person/name").string("june"));
    }
```

- XMLMessageConverter를 사용하려면 다양한 수동설정이 필요하다.
- 스프링 MVC에서는 Header정보가 중요하다.
