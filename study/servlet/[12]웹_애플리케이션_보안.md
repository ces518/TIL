# 지옥 스터디 - 12 웹 애플리케이션 보안

## 서블릿 보안 4요소
- 인증 (Authentication)
- 인가 (Authorization)
- 비밀 보장 (Confidentiality)
- 데이터 무결성 (Data Integrity)

## 컨테이너의 인증 방식
1. 사용자의 요청이 오면 컨테이너는 해당 요청이 **보안 테이블** 에 존재하는 URL 인지 확인한다
   1. 보안 테이블에 있는 URL 이라면 401 응답
2. 401 응답을 받고, 사용자가 인증 정보와 함께 재 요청
3. 사용자 인증 정보를 확인하고 **접근 권한** 을 확인 한 뒤 클라이언트 에게 정상 응답

> 컨테이너에서 지원하는 인증 방식은 **선언적인 방식** 이라고 표현한다.

`보안 영역`
- 서블릿 스펙 관점에서 보안 영역은, **인증 정보가 저장된 곳** 을 의미한다.
- 톰캣에서는 tomcat-users.xml
- 이는 특정 애플리케이션에만 적용되는 것이 아닌, 해당 톰캣에서 기동되는 모든 애플리케이션에 적용된다.
- 이런 방식을 **메모리 상주 보안 영역 정보 (Memory Realm)** 이라고 하는데, 톰캣 기동시 해당 파일을 읽어 메모리에 상주시키기 때문이다.
  - 역으로 생각해보면, 해당 정보가 변경될 경우 톰캣을 재기동해야 한다는 문제가 있다.

```xml
<tomcat-users>
   <role rolename="Guest"/>
   <role rolename="Member"/>
   <user name="Bill" password="coder" roles="Member, Guest"/>
</tomcat-users>
```

`롤 정의하기`
- 서블릿 환경에서 일반적인 인가방식은 특정 보안 역할을 가진 사용자만 특정 서블릿에 HTTP 요청을 날릴 수 있도록 설정하는 것
- 이는 user 파일에 정의한 역할 정보와 배포 서술자에 있는 보안 역할을 서로 매핑해야 하는것을 의미한다.

```xml
<!-- tomcat-users.xml -->
<tomcat-users>
   <role rolename="Admin"/>
   <role rolename="Member"/>
   <role rolename="Guest"/>
   <user username="Annie" password="admin" roles="Admin, Member, Guest"/>
</tomcat-users>
<!-- DD -->
<security-role><role-name>Admin</role-name></security-role>
<security-role><role-name>Member</role-name></security-role>
<security-role><role-name>Guest</role-name></security-role>
```

`자원/메소드 제약 정의하기`
- 주어진 자원/메소드 쌍을 특정 역할을 가진 사용자만 접근 가능하도록 설정하는 방법
- 중요한 것은 자원 단위가 아닌 **HTTP 요청 단위** 로 제약조건을 거는 것이다.
  - 만약 http-method 항목을 사용해 제약을 걸었다면, 명시된 메소드 외에는 제약조건이 걸리지 않는다.
  - ex) GET 을 명시했다면, GET 을 제외한 모든 메소드는 제약이 걸리지 않는점에 유의
  
```xml
<security-constraint>
  <web-resource-collection>
    <web-resource-name>UpdateRecipes</web-resource-name>
    <uri-pattern>/Beer/AddRecipe/*</uri-pattern>
    <uri-pattern>/Beer/ReviewRecipe/*</uri-pattern>
    <http-method>GET</http-method>
    <http-method>POST</http-method>
  </web-resource-collection>
  
  <auth-constraint>
    <role-name>Admin</role-name>
    <role-name>Member</role-name>
  </auth-constraint>
</security-constraint>
```

`auth-constraint 의 우선순위`
- role-name 이 각각 등록되어 있다면, 등록된 모든 역할이 접근 가능
- role-name 에 * 가 존재한다면, 모든 사용자가 접근 가능
- <auth-constraint/> 가 존재한다면 누구도 접근할 수 없다 (가장 마지막에 적용됨)
- auth-constraint 항목이 없다면 모든 사용자가 접근 가능

`Role Mapping`
- isUserInRole() 메소드를 이용해 서블릿에 특정 롤에 대한 인가 처리를 하드코딩 한 경우가 있다.
- A 회사에서는 Manager 라는 롤이 없을 수 있고, Admin 에 더 적합할 수 있는데, 이런 경우에 대비해 DD 를 통해 Role Mapping 기능을 제공한다.

```java
if (request.isUserInRole("Manager")) {
    // ....    
}
```

```xml
<servlet>
  <security-role-ref>
    <role-name>Manager</role-name>
    <role-link>Admin</role-link>
  </security-role-ref>
</servlet>

<security-role>
  <role-name>Admin</role-name>
</security-role>
```

> security-role 에 동일한 role-name 을 가진 role 이 존재하더라도, 컨테이너는 security-role-ref 에 존재하는 값을 먼저 참조한다는 점에 유의해야한다.

## 인증방식
- BASIC
  - HTTP 공식 인증 프로토콜
  - 사용자 이름과 비밀번호를 콜론으로 이어서 합치고, base-64 인코딩한다.
  - base-64 인코딩은 쉽게 디코딩이 가능하기때문에, 사실상 평문을 보내는것과 동일한 수준
- DIGEST
  - HTTP 공식 인증 프로토콜
  - 비밀번호를 절대 평문으로 전송하지 않는다.
  - 비밀번호를 보내는 대신, 비밀번호를 비가악적으로 뒤섞은 지문 또는 요약본을 보낸다.
- CLIENT-CERT
  - 로그인 정보를 공인키 인증 (PKC) 를 사용해 전송한다
  - 클라이언트가 인증서를 가지고 있어야 하기 때문에 비즈니스 환경에서 주로 사용한다.
- FORM
  - 일반적으로 많이 사용하는 방식
  - 기본적으로 암호화되지 않은 상태로 전송되기 때문에 SSL 을 적용해 사용함
  
`인증 방식 설정`

```xml
<login-config>
  <auth-method>BASIC</auth-method>
  <auth-method>DIGEST</auth-method>
  <auth-method>CLIENT-CERT</auth-method>
  <auth-method>FORM</auth-method>
  <form-login-config>
    <form-login-page>/loginPage.html</form-login-page>
    <form-error-page>/loginError.html</form-error-page>
  </form-login-config>
</login-config>
```

## 데이터 기밀성과 데이터 무결성 선언
- 데이터의 기밀성과 무결성을 보장하려면 DD 를 통해 설정이 가능하다.

```xml
<user-data-constraint>
  <transport-quarantee>CONFIDENTIAL</transport-quarantee>
</user-data-constraint>
```
- NONE
  - 기본 값; 데이터를 보호하지 않음
- INTEGRAL
  - 전송중 데이터가 변조되지 않음을 보장
- CONFIDENTIAL
  - 전송중 그 누구도 데이터를 훔쳐보지 않음을 보장

> INTEGRAL, CONFIDENTIAL 모두 SSL 이 적용되어 있어야 해당 보장이 가능하며, SSL 적용이 되어 있지않다면 NONE 과 다름없다.

