# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 회원 기능 테스트

#### 회원기능 테스트
`테스트 요구사항`
- 회원가입을 성공해야 한다.
- 회원가입 시 같은 이름이 있다면 예외가 발생해야 한다.

`회원가입 테스트 코드`
```java
/**
 * 단위 테스트 보다는 외부 DB에서 잘 동작하는지 확인하기 위해
 * 통합 테스트로 진행한다.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class MemberServiceTest {

    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;

    @Test
    //@Rollback(false)
    public void 회원가입 () throws Exception {
        // given
        Member member = new Member();
        member.setName("PARK");

        // when
        Long joinedId = memberService.join(member);

        // then
        assertThat(member).isEqualTo(memberRepository.find(joinedId));
        // join 을 통해서 생성된 멤버와, memberRepository를 통해 찾아온 member가 같아야한다.
        // JPA 는 같은 트랜잭션내에서 같은 식별자를 가진 엔티티는 동일성을 보장한다.
    }
}
```

- 해당 테스트 코드에서는 INSERT 쿼리가 나가지 않는다.
    - @Transacitonal은 테스트에서는 기본적으로 Rollback 이 되기 떄문이다.
    - 즉 영속성 컨텍스트에서 flush 되지 않는다.
    - 만약 Rollback 되는걸 바라지 않고, 데이터가 제대로 들어갔는지 확인하고 싶다면 @Rollback(false) 를 사용할것
    - 또는 **EntityManager를 주입받아 강제적으로 flush 시켜주면 된다.**

> JPA의 같은 트랜잭션내에서 같은 식별자를 가진 엔티티의 동일성을 보장함을 이용해 assertion을 한다.

`중복회원 가입시 예외발생에 대한 테스트 코드`
```java
// try - catch 방식보다 깔끔한 예외발생 테스트기능
// expected 를 활용한다.
@Test(expected = IllegalStateException.class)
public void 중복_회원_예외 () throws Exception {
    // given
    Member member = new Member();
    member.setName("PARK");

    Member member2 = new Member();
    member2.setName("PARK");

    // when
    memberService.join(member);
    // 같은 이름으로 가입을 시도했기 때문에 예외가 발생해야 한다.
    // 예외 발생후 아래 라인으로 이동하면 안된다.
    memberService.join(member2);

    /*
    try {
        memberService.join(member2);
    } catch (IllegalStateException e) {
        // try-catch 를 활용해 예외가 발생하면 메소드를 종료시켜버린다.
        // 하지만 이러한 방식은 코드가 지저분해진다.
        return;
    }
    */

    // then
    /**
        * 만약 여기 코드라인까지 오게된다면 성공이라고 뜨게 된다.
        * 그럼 잘못된 테스트 코드이다.
        * 이를 위해 junit에서 fail 이라는 메소드를 제공함.
        * 테스트를 실패시킨다.
        */
    fail("예외가 발생해야함");
}
```

- 중복된 이름의 회원으로 가입 시도시 IllegalStateException 이 발생한다.
- 테스트 코드에서 이에 대한 아무런 처리를 하지 않는다면 테스트가 실패하거나, 잘못된 테스트 결과를 초례할 수 있다.
- 위의 테스트 코드에서 중복된 join 코드 라인을 모두 지나치고 다음 코드라인으로 이동한다면 잘못된 테스트 코드이다.
    - 이를 위해 junit 에서 **fail()** 메소드를 제공한다.
    - 만약 해당 코드라인으로 온다면 테스트 케이스를 실패 시켜버린다.

- IllegalStateException이 발생하였을때, try-cathc를 활용해 예외처리를 하여 다음 코드라인으로 이동하지않고, 테스트를 성공 시킨다면
- 정상적인 테스트 코드이긴 하나 코드가 난잡해진다.
    - 지금은 간단한 테스트코드 이지만 테스트가 복잡해 질수록 점점 더 난잡해질것이다.
- @Test 애노테이션의 expected 속성에 이 테스트케이스에서 발생할 예외의 종류를 정의해 둔다면
- 해당 테스트케이스에서 예외가 발생하였을때 정의해둔 타입이라면 테스트가 성공처리 된다.

> 위의 방법 외에도 실패에 대한 테스트 케이스를 작성하는 다양한 방법이 존재한다. 
- https://github.com/ces518/ILE/blob/master/unit-test/Junit%20-%20Exception%20Test.md 참고


#### 격리된 테스트 환경
- 지금의 테스트 환경은 문제가 있다.
- 현재는 실제 외부에 존재하는 디비를 사용했다.
- 이러한 테스트 코드를 병렬로 여러개 돌렸을때의 문제 등 이 있다.
- 테스트 종료시 데이터가 초기화 되지 않는다.

> 테스트 시 마다 인메모리 디비를 사용해 완전히 격리된 환경으로 테스트를 진행하는 것이 좋다.

- Test 디렉터리에 resources디렉터리를 생성하고, yml 파일을 생성해준다.
- 기존의 datasource 설정을 모두 제거한다.
```yml
#spring:
#  datasource:
#    url: jdbc:h2:tcp://localhost/./jpashop # MVCC=TRUE 옵션을 주면 여러곳에서 접근했을때 좀더 빠르다. 개발시엔 크게 상관이없음
#    username: jpashop
#    password: jpa
#    driver-class-name: org.h2.Driver # H2 JDBC
#
#  jpa:
#    hibernate:
#      ddl-auto: create # DDL CREATE MODE, 애플리케이션 실행시 지우고 새롭게 생성한다.
#    properties:
#      hibernate:
##        show_sql: true # SQL show (System.out 사용 사용하지 말것)
#        format_sql: true # SQL formatting
#    database-platform: org.hibernate.dialect.H2Dialect

logging:
  level:
    org.hibernate.SQL: debug # Hibernate Logging LEVEL 설정, Hibernate SQL이 모두 보이게 된다. (Logger 사용)
    org.hiberate.type: trace
```

> Spring Boot 는 기본적으로 datasource 설정이 존재하지 않는다면 h2 인메모리 설정을 가져간다.


* 운영에서의 환경과 테스트에서의 환경은 격리해야 한다, 변경점이 서로 다르기 때문에 두 환경이 항상 같을 수 없기 때문이다.
