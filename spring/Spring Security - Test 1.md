# Spring Security - Test 1
- 기존의 테스트 방법
    - 애플리케이션을 실행
    - 웹 브라우저에서 직접 확인
    - 코드가 변경될때마다 유저를 다시 생성해주고, url 하나하나에 일일히 접근해야한다.
    - 매우 번거롭다
- 소스코드를 사용해서 테스트를 진행한다.

#### 의존성 추가
- spring-security-test 는 version관리가 되고있지 않다.
- 따라서 버전을 명시해주는것이 좋다.
```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <version>${spring-security.version}</version>
    <scope>test</scope>
</dependency>
```

#### 테스트 코드
- JUnit을 활용하여 index 페이지에 대한 테스트 진행하는 매우 간단한 코드이다.
- 하지만 이때 사용자의 인증 정보를 TEST 하려면 어떻게 해야 할까 ?
- spring-security-test 에서 제공하는 with method를 활용하자.
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class AccountControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void index_anonymous () throws Exception {
        mockMvc.perform(get("/")
                    .with(anonymous())
                ) // 인증되지 않은 유저
                .andDo(print())
                .andExpect(status().isOk());
    }

    @Test
    public void index_user () throws Exception {
        mockMvc.perform(get("/")
                   .with(user("june").roles("USER"))
                    // 유저 인증 정보를 Mocking 하고 테스트를 진행한다 (데이터베이스에는 존재하지않음. 로그인한 상태라고 가정)
                )
                .andDo(print())
                .andExpect(status().isOk());
    }

}
```


with() method를 이용해 유저 인증정보를 Mocking 할 수 있다.
이 때 해당 유저가 데이터베이스 존재하는것이 아닌, 로그인을 한 상태라고 가정하는것이다.
결코 데이터베이스에 해당 유저가 존재한다는 의미가 아니다.

아래 코드를 살펴보며 자세히 알아보자.
with() method를 이용해 유저 인증정보를 Mocking 한다.
- anonymous() 익명 사용자
- user("유저명").password("패스워드").roles("권한") 이다.
    - password는 의미가 없기때문에 유저명과 , 권한만 Mocking하도록 하자.
```java
.with(anonymous())
.with(user("june").roles("USER"))
```


#### 테스트 결과
- index_anonymous 테스트 결과
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /
       Parameters = {}
          Headers = []
             Body = null
    Session Attrs = {}

Handler:
             Type = me.june.springsecurity.form.SampleController
           Method = public java.lang.String me.june.springsecurity.form.SampleController.index(org.springframework.ui.Model,java.security.Principal)

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = index
             View = null
        Attribute = message
            value = Hello Spring Security

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Language:"en", Content-Type:"text/html;charset=UTF-8", X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", Cache-Control:"no-cache, no-store, max-age=0, must-revalidate", Pragma:"no-cache", Expires:"0", X-Frame-Options:"DENY"]
     Content type = text/html;charset=UTF-8
             Body = <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Welcome Page</title>
</head>
<body>
    <h1>Hello Spring Security</h1>
</body>
</html>

    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

- index_user 테스트 결과
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /
       Parameters = {}
          Headers = []
             Body = null
    Session Attrs = {SPRING_SECURITY_CONTEXT=org.springframework.security.core.context.SecurityContextImpl@ca54ce9: Authentication: org.springframework.security.authentication.UsernamePasswordAuthenticationToken@ca54ce9: Principal: org.springframework.security.core.userdetails.User@31f442: Username: june; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Granted Authorities: ROLE_USER; Credentials: [PROTECTED]; Authenticated: true; Details: null; Granted Authorities: ROLE_USER}

Handler:
             Type = me.june.springsecurity.form.SampleController
           Method = public java.lang.String me.june.springsecurity.form.SampleController.index(org.springframework.ui.Model,java.security.Principal)

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = index
             View = null
        Attribute = message
            value = Hello Indexjune

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Language:"en", Content-Type:"text/html;charset=UTF-8", X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", Cache-Control:"no-cache, no-store, max-age=0, must-revalidate", Pragma:"no-cache", Expires:"0", X-Frame-Options:"DENY"]
     Content type = text/html;charset=UTF-8
             Body = <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Welcome Page</title>
</head>
<body>
    <h1>Hello Indexjune</h1>
</body>
</html>

    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

두 테스트 모두 성공하는것을 확인 할 수 있다.

그렇다면 이번에는 admin page에 대한 테스트를 진행 해보도록 하자.

#### ADMIN Page 테스트 코드
- /admin 으로 USER 권한을 가지고 있는 사용자가 요청을 하면 403 응답을 바라고, 
- /admin 으로 ADMIN 권한을 가지고 있는 사용자가 요청을 하면 200 정상적인 응답을 바라는 테스트 코드이다.
```java
@Test
public void admin_user () throws Exception {
    mockMvc.perform(get("/admin")
                .with(user("june").roles("USER")))
            .andDo(print())
            .andExpect(status().isForbidden());
}


@Test
public void admin_admin () throws Exception {
    mockMvc.perform(get("/admin")
                .with(user("admin").roles("ADMIN")))
            .andDo(print())
            .andExpect(status().isOk());
}
```

테스트는 예상한대로 모두 성공 한다.

- admin_user 테스트 결과
```java
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /admin
       Parameters = {}
          Headers = []
             Body = null
    Session Attrs = {SPRING_SECURITY_CONTEXT=org.springframework.security.core.context.SecurityContextImpl@ca54ce9: Authentication: org.springframework.security.authentication.UsernamePasswordAuthenticationToken@ca54ce9: Principal: org.springframework.security.core.userdetails.User@31f442: Username: june; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Granted Authorities: ROLE_USER; Credentials: [PROTECTED]; Authenticated: true; Details: null; Granted Authorities: ROLE_USER}

Handler:
             Type = null

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
           Status = 403
    Error message = Forbidden
          Headers = [X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", Cache-Control:"no-cache, no-store, max-age=0, must-revalidate", Pragma:"no-cache", Expires:"0", X-Frame-Options:"DENY"]
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

하지만 여기서 문제점이 하나 있다.
바로 User를 Mocking하는 코드가 테스트 코드내에 존재한다는 것이다.
지금은 매우 간단한 테스트 코드이지만. 테스트 코드가 길어진다면 ? (요청헤더, 응답헤더, 파라메터, Assertion 등..)
어떤 User 정보를 Mocking하는지 알아보기가 힘들어 진다.

spring-security-test가 제공하는 애노테이션을 사용하도록 하자.

#### @WithAnnoymousUser @WithMockUser
- @WithAnnoymousUser
    - 이름 그대로 익명 사용자를 Mocking하는 애노태이션이다.
- @WithMockUser(username = "사용자 이름", roles = "권한")
    - 사용자를 Mocking하는 애노테이션이다.
    - username: 사용자 이름
    - roles: 권한

애노테이션을 적용하여 기존 테스트코드들을 수정해보자
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class AccountControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    @WithAnonymousUser
    public void index_anonymous () throws Exception {
        mockMvc.perform(get("/")) // 인증되지 않은 유저
                .andDo(print())
                .andExpect(status().isOk());
    }


    @Test
    @WithMockUser(username = "june", roles = "USER")
    public void index_user () throws Exception {
        mockMvc.perform(get("/"))
                .andDo(print())
                .andExpect(status().isOk());
    }


    @Test
    @WithMockUser(username = "june", roles = "USER")
    public void admin_user () throws Exception {
        mockMvc.perform(get("/admin"))
                .andDo(print())
                .andExpect(status().isForbidden());
    }


    @Test
    @WithMockUser(username = "admin", roles = "ADMIN")
    public void admin_admin () throws Exception {
        mockMvc.perform(get("/admin"))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```

각 테스트 코드가 어떤 유저정보를 Mocking하는지 알아보기 쉽게 개선되었다.
하지만 User를 Mocking하는 코드 중복이 발생하였다.
```java
@WithMockUser(username = "june", roles = "USER")
```

#### @WithUser
- 특정 애노테이션을 사용하는데 있어서 중복이 발생하는 경우 메타애노테이션을 만들어 이를 해결할 수 있다.
```java
@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(username = "june", roles = "USER")
public @interface WithUser {

}
```

다음은 @WithUser 애노테이션을 활용해 중복을 제거한 코드이다.
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class AccountControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    @WithAnonymousUser
    public void index_anonymous () throws Exception {
        mockMvc.perform(get("/")) // 인증되지 않은 유저
                .andDo(print())
                .andExpect(status().isOk());
    }


    @Test
    @WithUser
    public void index_user () throws Exception {
        mockMvc.perform(get("/"))
                .andDo(print())
                .andExpect(status().isOk());
    }


    @Test
    @WithUser
    public void admin_user () throws Exception {
        mockMvc.perform(get("/admin")
                    .with(user("june").roles("USER")))
                .andDo(print())
                .andExpect(status().isForbidden());
    }


    @Test
    @WithMockUser(username = "admin", roles = "ADMIN")
    public void admin_admin () throws Exception {
        mockMvc.perform(get("/admin")
                    .with(user("admin").roles("ADMIN")))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```
