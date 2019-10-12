# Spring Security - Method Security
- SpringSecurity는 Web기반의 Security 외에 Method Security 기능을 제공한다.
- 서비스 계층을 직접 호출할때 사용할 수 있는 보안 기능이다.
- Web기반의 Security를 적용했을때는 어울리지 않는 기능이다.
- Web 애플리케이션 이 외에도 데스크탑 애플리케이션 사용시 적용이 가능하다.

#### @EnableGlobalMethodSecurity
- MethodSecurity는 우리가 기존에 사용했던 SecurityConfig 설정이 적용되지 않는다.
- MethodSecurity용 설정이 따로 필요한데 이때 사용하는것이 **@EnableGlobalMethodSecurity**이다.
```java
@Configuration
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true, jsr250Enabled = true)
public class MethodSecurity {
}
```

#### @EnableGlobalMethodSecurity 의 속성들
- securedEnabled, prePostEnabled, jsr250Enabled 3개의 옵션이 존재한다.
- 이름만 봤을때 어떠한 설정을 활성화 시키는 옵션이라는 느낌을 준다.

1.securedEnabled
- @Secured 애노테이션을 사용하여 인가 처리를 하고 싶을때 사용하는 옵션이다.
- 기본값은 false

2.prePostEnabled
- @PreAuthorize, @PostAuthorize 애노테이션을 사용하여 인가 처리를 하고 싶을때 사용하는 옵션이다.
- 기본값은 false

3.jsr250Enabled
- @RolesAllowed 애노테이션을 사용하여 인가 처리를 하고 싶을때 사용하는 옵션이다.
- 기본값은 false


##### @Secured, @RolesAllowed
- 특정 메서드 호출 이전에 권한을 확인한다.
- SpEL 지원하지 않는다.
- @Secured 는 스프링에서 지원하는 애노테이션이며, @RolesAllowed는 자바 표준
```java
@Secured("ROLE_USER")
@RolesAllowed("ROLE_USER")
public void dashboard () {
        ...
}
```
##### @PreAuthorize, @PostAuthorize
- 특정 메서드 호출 전, 후 이전에 권한을 확인한다.
- SpEL을 지원한다.
- 스프링에서 지원하는 애노테이션이다.
- PostAuthorize는 해당 메서드의 리턴값을 **returnObject** 로 참조하여 SpEL을 통해 인가 처리를 할 수 있다.

```java
@PreAuthorize("hasRole('USER')")
@PostAuthorize("hasRole('USER')")
@PostAuthorize("returnObject.username == authntication.principal.nickName")
public void dashboard () {
    ...
}
```

#### MethodSecurity의 RoleHierachy
- 우리가 Web에서 Role의 계층구조 설정을 해주었지만 MethodSecurity에서는 적용 되지 않는다.
- MethodSecurity용 RoleHierachy 설정을 따로 해주어야한다.
```java
@Configuration
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true, jsr250Enabled = true)
public class MethodSecurity extends GlobalMethodSecurityConfiguration {

    /* MethodSecurity용 Hierachy 설정을 해주어야함 */
    @Override
    protected AccessDecisionManager accessDecisionManager() {
        RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
        roleHierarchy.setHierarchy("ROLE_ADMIN > ROLE_USER");
        AffirmativeBased accessDecisionManager = (AffirmativeBased) super.accessDecisionManager();
        accessDecisionManager.getDecisionVoters().add(new RoleHierarchyVoter(roleHierarchy));
        return accessDecisionManager;
    }
}
```

- 먼저 GlobalMethodSecurityConfiguration 클래스를 상속받아 accessDecisionManager method를 Overried 한다.
- 다음으로 WebSecurity 설정에서 해준것과 마찬가지로 RoleHierarachy를 설정한뒤 accessDicisionVoter에 추가해준다.

#### 정리
- Spring Security에서는 특정 메서드에 권한 처리를 하는 MethodSecurity 기능을 제공한다.
- MethodSecurity는 WebSecurity와는 별개로 동작하기 때문에 추가적인 설정이 필요하다.
- 주로 데스크탑 애플리케이션에서 사용한다.

#### 참조
- https://docs.spring.io/spring-security/site/docs/5.1.5.RELEASE/reference/htmlsingle/#jc-method
- https://docs.spring.io/spring-security/site/docs/3.0.x/reference/el-access.html
- https://www.baeldung.com/spring-security-method-security
