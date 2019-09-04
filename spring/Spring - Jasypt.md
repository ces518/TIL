# Spring - Jasypt
#### 개요
- Project의 DB 커넥션 정보, API 토큰 등 민감한 정보들이 Properties 파일에 그대로 노출되어 있음
- LX 같은 경우 해당 Property 정보 암호화 요구 사례 존재

#### Jasypt
- Java 기반 암호화를 손 쉽게 제공하는 라이브러리
- 대부분 예제들이 Java 기반임.
- Spring과 연동하여 손쉽게 사용하고 싶음
    - 원하는 방식의 라이브러리가 아니다.

#### Spring - Jasypt
- Spring 에 적용가능한 Jasypt 라이브러리
- 암호화 된 패스워드를 풀기위한 Key는 서버의 환경변수 등에 보관하는것이 안전함

#### 의존성
- Jasypt를 스프링 부트에서 편하게 사용가능한 의존성
- Version 마다 상이하며 최소 버전은 JDK8 이상임
- JDK7 환경에서 사용하려면 버전에 java7이 명시되어있는 버전을 사용해야한다.
```xml
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>1.4-java7</version>
</dependency>
```

#### 설정
- StandardPBEStringEncryptor를 Bean으로 등록 해 주어야한다.
- Properties 파일에서 해당 암호화의 키값 주입받는다.
    - @Value("${encrypt.password}") String password 
```java
@Configuration
public class EnvConfig {

    @Bean
    public StandardPBEStringEncryptor standardEncryptor (@Value("${encrypt.password}") String password) {
        StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
        EnvironmentPBEConfig config = new EnvironmentPBEConfig();
        config.setPassword(password);
        config.setAlgorithm("PBEWITHMD5ANDDES");
        encryptor.setConfig(config);
        return encryptor;
    }
}
```

#### 암호화 방법
- StandardPBEStringEncryptor을 Bean으로 주입받아 사용해야한다.
    - Spring IoC Container 가 관리하는 빈으로 등록 하였음.
- stringEncryptor.encrypt(); 메서드로 암호화
- stringEncryptor.decrypt(); 메서드로 복호화가 가능하다.
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class EnvConfigTest {

    @Autowired
    StandardPBEStringEncryptor stringEncryptor;

    @Test
    public void test () {
        String new_mec_core = stringEncryptor.encrypt("NEW_MEC_CORE");
        String decrypt = stringEncryptor.decrypt(new_mec_core);
        System.out.println(new_mec_core);
    }
}
```

#### 설정파일에 실 적용방법
- 암호화된 문자열을 ENC() 로 감싸주어 암호화된 문자열이라고 알려주어야한다.
- 해당 wrapping 이 존재하지 않을경우 암호화된 문자열로 인식하지못해 복호화를 진행하지 않음.
- 모든 properites 값들에 적용이 가능하다.
```yml
spring:
  datasource:
    url: ENC(hbKFgcxxQmmD5L+hKOROMdqRHSWhNEgGOuUhPupXizvcpzhR44d2t0xkfuqxjvo4n5HnwKZKkLJPxtF6tnG8qYPVIzZU7YiuD/5JYGZiLe15+JUm4aG3BBGf2LktyUYeccmzN1AsBnQMnaGkF2QqMNdAJh8D2wXRLmhKegALb6PfRQawoPMAyI4IeO9tnXCO)
    username: ENC(jo9UXusdaPzfGHRdET/2eqzs835TIJix)
    password: ENC(j4K/SU7rTg83bPwG1Kdrd4/u3BfjyqMj)
    driver-class-name: net.sf.log4jdbc.DriverSpy
    tomcat:
      max-active: 400
      max-wait: 10000
      max-idle: 20
      test-on-borrow: true
      validation-query: select 1
  http:
    multipart:
      max-file-size: 200MB
      max-request-size: 200MB
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
```

#### 공식문서
- http://www.jasypt.org/
