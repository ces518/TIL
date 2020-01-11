# JPA 기본편 - Named 쿼리

#### JPQL - Named 쿼리
- 미리 정의해 두고 사용하는 JPQL
    - 쿼리에 이름을 부여해서, Named쿼리라고 부른다.
- 정적 쿼리
- 어노테이션, XML 에 정의한다.
- 애플리케이션 로딩 시점에 초기화 후 재사용 가능하다.
    - **애플리케이션 로딩 시점에 쿼리를 검증한다.**
    - 휴먼에러를 1차적으로 방지할 수 있다.

> JPA가 Named 쿼리를 미리 파싱해두고 캐시해 두기때문에 코스트가 발생하지 않는다.

#### 애노테이션을 활용한 정의
```java
@Entity
@NamedQuery(name = "Member.findByUsername",
            query = "select m from Member m where m.username = :username"
)
public class Member {
    ...
}
```

- @NamedQuery 라는 애노테이션을 사용해서 정의한다.
- name
    - NamedQuery의 이름 (별칭)
    - 관례는 엔티티명.쿼리명 으로 정의한다.
- query
    - 실행될 JPQL

#### Named 쿼리 환경에 따른 설정
- XML 이 항상 우선권을 가진다.
- 애플리케이션 운영환경에 따라 다른 XML을 배포할 수 있다.

> 솔루션을 개발해서 배포할때 특정 상황마다 다른 쿼리가 나가야한다면, XML을 이용해 다르게 배포할 수 있다.

#### Tip
- Spring Data JPA 에서는 @Query 를 이용해 바로 사용 할 수 있다.
- 이름이 없는 NamedQuery로 등록해 버린다.
- 애플리케이션 로딩시점에 파싱해서 문법오류를 다 잡아준다.

> Query 를 엔티티에 등록해서 사용하는것은 좋지 않은 방법이다. 엔티티가 지저분해짐..
