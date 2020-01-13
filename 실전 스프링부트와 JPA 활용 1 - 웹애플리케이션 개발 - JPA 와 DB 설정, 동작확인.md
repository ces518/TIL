# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - JPA 와 DB 설정, 동작확인

#### JPA - H2 설정하기
```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/./jpashop # MVCC=TRUE 옵션을 주면 여러곳에서 접근했을때 좀더 빠르다. 개발시엔 크게 상관이없음
    username: jpashop
    password: jpa
    driver-class-name: org.h2.Driver # H2 JDBC

  jpa:
    hibernate:
      ddl-auto: create # DDL CREATE MODE, 애플리케이션 실행시 지우고 새롭게 생성한다.
    properties:
      hibernate:
#        show_sql: true # SQL show (System.out 사용 사용하지 말것)
        format_sql: true # SQL formatting
    database-platform: org.hibernate.dialect.H2Dialect
```

#### Member 엔티티 생성, Member Repository 생성/테스트 하기
```java
@Entity
@Getter @Setter
public class Member {

    @Id @GeneratedValue
    private Long id;
    private String username;
}

@Repository
public class MemberRepository {

    // @Autowired 를 사용해도 된다
    // @PersistenceContext 가 JPA의 표준
    // Spring Boot AutoConfigure 로 인해 자동 설정 되어있다.
    @PersistenceContext
    private EntityManager em;

    public Long save (Member member) {
        // 저장을 한뒤 member를 반환하지않고, id만 반호나함
        // 커맨드와 쿼리를 분리하라 원칙
        // 저장하는것은 가급적이면 사이드 이펙트를 일으키는 커맨드성이기 때문에 리턴값을 주지 말것.
        em.persist(member);
        return member.getId();
    }

    public Member find (Long id) {
        return em.find(Member.class, id);
    }
}

@RunWith(SpringRunner.class)
@SpringBootTest
public class MemberRepositoryTest {

    @Autowired MemberRepository memberRepository;

    @Test
    @Transactional // 테스트 종료후 롤백
    @Rollback(false) // 롤백시키고 싶지 않을때 사용
    public void save () throws Exception {
        // given
        Member member = new Member();
        member.setUsername("memberA");

        // when
        Long saveId = memberRepository.save(member);
        Member findMember = memberRepository.find(saveId);

        // then
        assertThat(findMember.getId()).isEqualTo(saveId);
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        assertThat(findMember).isEqualTo(member);
        // 같은 트랙잭션내 (같은 영속성 컨텍스트) 에서 식별자가 같다면 같은 엔티티임을 보장한다.
        // select 쿼리 자체가 나가지않음
        System.out.println("(findMember == member) = " + (findMember == member));
    }
}
```

- EntityManager 를 의존성 주입받을때 스프링 애노테이션인 @Autowired를 사용해도 된다.
- @PersistenceContext 는 JPA 표준 애노테이션이다.

`커맨드와 쿼리를 분리하라`
- 커맨드 ( Create - Insert, Update, Delete : 데이터를 변경) 와 쿼리 ( Select - Read : 데이터를 조회)의 책임을 분리한다는 것이다.
- persistence 영역을 통해 저장을 한뒤, member를 직접 반환하지 말것.
- 저장하는 행위는 사이드 이벡트를 일으키는 커맨드성 이기 떄문에 될수 있으면 리턴값을 주지 말것
- 참고
    - https://www.popit.kr/cqrs-eventsourcing/
    - https://github.com/jojoldu/review/blob/master/SpringCamp_2017_2%EB%B6%80/README.md


#### 쿼리 파라미터 로그 남기기
- 1.로그에 다음 속성을 추가한다.
```yml
logging:
  level:
    org.hibernate.SQL: debug # Hibernate Logging LEVEL 설정, Hibernate SQL이 모두 보이게 된다. (Logger 사용)
    org.hiberate.type: trace
```
- 2.p6spy
    - 다음과 같이 원본 SQL과 바인딩된 파라메터 정보가 보인다.
```java
insert into member (username, id) values (?, ?)
insert into member (username, id) values ('memberA', 1);
```

- 참고
    - https://github.com/gavlyukovskiy/spring-boot-data-source-decorator

> 운영 에서는 성능 테스트를 진행후 사용할 것을 권장

#### gradle 설치
- https://blusky10.tistory.com/234
- 압축파일 이용해 설치후 환경변수 등록
- 환경변수 파일 열기
    - open ~/.bash_profile
- 환경변수 등록
    - export PATH=$PATH:설치경로/bin
- 환경변수 파일 적용
    - source ~/.bash_profile
