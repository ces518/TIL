# REST API - Security - Account 도메인 추가
- 현재 API는 문제가 존재함 
    - 인증 절차가 존재하지 않는다.
    - 이벤트와 연관이있는 Account 를 가지는 사람만 수정 삭제가 가능해야하는데, 현재는 누구자 수정, 삭제가 가능함.
- Spring Security OAuth2 적용
    - password 라는 Grant Type 을 사용하여 적용
    - Account 도메인 필요

#### Account
- Account 도메인
    - id: 식별자
    - email: 이메일
    - password: 패스워드
    - roles: 권한

- fetchType 을 EAGER로 설정한 이유 ?
    - 1:N 관계 이기때문에 기본 fetch Type은 Lazy 이다.
    - 하지만 Role의 개수가 적기도하고, 매번 권한이 필요하기 때문에 EAGER Fetch 사용
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @NoArgsConstructor @AllArgsConstructor
public class Account {

    @Id @GeneratedValue
    private Integer id;

    private String email;

    private String password;

    /* 기본값이 Lazy 이지만 매번 권한이 필요하기때문에 EAGER */
    @ElementCollection(fetch = FetchType.EAGER)
    @Enumerated(EnumType.STRING)
    private Set<AccountRole> roles;
}
```
#### AccountRole
- ADMIN, USER 권한 ENUM
```java
public enum AccountRole {
    ADMIN, USER
}
```
