# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 회원 서비스 개발

#### 회원 서비스 개발
```java
@Service
@RequiredArgsConstructor
/**
 * JPA에서 데이터조작이 발생하는것은 트랜잭션 내에서 이루어 져야함
 * @Transational을 사용해야한다.
 * Class Level에 선언하면 public 메소드에 모두 적용된다.
 * javax, spring 애노테이션 두가지가 존재함.
 * spring 애노테이션을 추천, 사용가능한 옵션들이 많다.
 *
 * readOnly=true 옵션을 주면, JPA에서 성능 최적화를 해준다.
 */
@Transactional(readOnly = true)
public class MemberService {

    /**
     * @Autowired 를 이용한 필드 인젝션보단 생성자를 활용한 주입을 추천함 (순환참조시 에러로 알려준다.)
     */
    private final MemberRepository memberRepository;

    /**
     * setter를 활용한 인젝션
     * 테스트코드 작성시 mock객체를 주입할수 있다는 장점이 있다.
     * 런타임 도중에 변경될 우려가있음 (치명적임)
     * @param memberRepository
     */
    /*
    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    */


    /**
     * 회원가입
     * @param member
     * @return
     */
    @Transactional
    public Long join (Member member) {
        // 이름이 같은 회원이 존재하는지 검증
        validateDuplicateMember(member);

        // save 하는시점에 @GenerateValue 를 사용중이라면 영속성컨텍스트에 영속화되는순간에 ID값을 세팅해준다.
        // 그래서 세이브후 Id가 있다는것이 보장된다.
        memberRepository.save(member);
        return member.getId();
    }

    /**
     * 회원전체 조회
     * @return
     */
    public List<Member> findMembers () {
        return memberRepository.findAll();
    }

    /**
     * 단건 조회
     * @param memberId
     * @return
     */
    public Member findOne (Long memberId) {
        return memberRepository.find(memberId);
    }

    /**
     * 중복회원 검증
     * @param member
     */
    private void validateDuplicateMember (Member member) {
        /**
         * 검증을 한다고해도 동시에 접근하면 문제가 될 수 있다.
         * 데이터베이스에 유니크 제약조건을 거는것을 권장함
         */
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if (!findMembers.isEmpty()) {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        }
    }
}
```

> JPA에서 데이터조작이 일어나는행위는 반드시 트랜잭션 내부에서 이루어 져야한다.

`@Transactional`
- JPA에서 데이터조작이 발생하는것은 트랜잭션 내에서 이루어 져야함
- @Transactional을 사용해야한다.
- Class Level에 선언하면 public 메소드에 모두 적용된다.
- javax, spring 애노테이션 두가지가 존재함.
- spring 애노테이션을 추천, 사용가능한 옵션들이 많다.
- readOnly=true 옵션을 주면, JPA에서 성능 최적화를 해준다.

> 비즈니스 로직에서 검증을 하더라도 동시성 문제가 발생할수 있기 떄문에 중복체크 같은 경우에는 데이터베이스에서 유니크 제약조건을 걸어두는걸 권장
