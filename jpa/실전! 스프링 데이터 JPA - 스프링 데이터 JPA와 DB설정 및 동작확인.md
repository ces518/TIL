# 실전! 스프링 데이터 JPA - 스프링 데이터 JPA와 DB설정 및 동작확인

#### 스프링 데이터 JPA와 DB설정 및 동작확인
- properties 대신 yml을 사용한다.
```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/datajpa
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        # show_sql: true
        format_sql: true

logging.level:
  org.hibernate.SQL: debug
# org.hibernate.type: trace
```

`Member`
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id @GeneratedValue
    private Long id;
    private String username;

    public Member(String username) {
        this.username = username;
    }
}
```

> Entity는 Setter는 제한하고 꼭 필요하다면 의미있는 비즈니스 메서드를 정의해서 사용하는것이 좋다.

#### 기존 방식의 리포지토리
```java
@Repository
public class MemberJpaRepository {

    @PersistenceContext
    private EntityManager em;

    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    public Member find(Long memberId) {
        return em.find(Member.class, memberId);
    }
}

// Junit 5 부터는 @RunWith 애노테이션이 더이상 필요없다.
@SpringBootTest
@Transactional
@Rollback(false)
class MemberJpaRepositoryTest {

    @Autowired MemberJpaRepository memberJpaRepository;

    /**
     * No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call 예외 발생
     * JPA의 모든 데이터조작은 트랜잭션 내에서 이루어 져야함
     */
    @Test
    public void testMember() {
        // JPA 특성상 동일 트랜잭션 내에서는 같은 식별자를 가진 엔티티는 동일성을 보장한다.
        Member member = new Member("memberA");
        Member savedMember = memberJpaRepository.save(member);

        Member findMember = memberJpaRepository.find(savedMember.getId());

        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        assertThat(findMember).isEqualTo(member);
    }
}
```

- Junit5 부터는 @RunWith 애노테이션이 필요없다.
- JPA의 모든 데이터 조작은 트랜잭션 내에서 이루어 저야하기 때문에 @Transactional 애노테이션이 없다면 예외가 발생한다.

#### Spring data JPA 
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}

@SpringBootTest
@Transactional
@Rollback(false)
class MemberRepositoryTest {

    @Autowired MemberRepository memberRepository;

    @Test
    public void testMember() {
        Member member = new Member("memberA");
        // JpaRepository의 상위 인터페이스에서 제공을한다.
        // -> 기본 CRUD, Paging 기능 제공
        Member savedMember = memberRepository.save(member);

        Member findMember = memberRepository.findById(savedMember.getId()).get();

        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        assertThat(findMember).isEqualTo(member);
    }
}
```
- Spring data JPA 를 사용하면 JpaRepository 인터페이스를 상속받는것만으로 CRUD, Paging기능까지 모두 제공 한다.

#### p6spy
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.7'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}
```

- SQL Log에서 실행된 쿼리를 확인하는데 어떤 값이 들어갔는지 확인하고 싶을 때가 있다.
- p6spy를 사용하면 쿼리에서 사용된 ?,? 와 같은 파라메터에 바인딩 된 값까지 확인이 가능하다.

```java
insert into member (username, id) values (?, ?)
insert into member (username, id) values ('memberA', 1);
```
