# 실전! 스프링 데이터 JPA - 도메인 클래스 컨버터

#### 도메인 클래스 컨버터
```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;

    @PostConstruct
    public void init() {
        memberRepository.save(new Member("userA"));
    }

    @GetMapping("/members/{id}")
    public String findMember(@PathVariable Long id) {
        Optional<Member> optionalMember = memberRepository.findById(id);
        return optionalMember.get().getUsername();
    }

    /*
        위의 API와 동일한 컨버팅 하는 기능을 Spring data JPA에서 제공한다.
        > 이런 도메인 클래스 컨버터는 매우 간단한 케이스에서만 사용이 가능하다.
        - 리포지토리를 사용해서 엔티티를 찾는다.
        도메인 클래스 컨버터로 엔티티를 파라메터로 받으면 단순 조회용으로만 사용해야한다.
        -> 트랜잭션 범위가 없는 케이스에서 조회 했기 때문에 엔티티를 수정하여도 DB에 반영되지 않음
     */
    @GetMapping("/members2/{id}")
    public String findMember2(@PathVariable("id") Member member) {
        return member.getUsername();
    }

}
```

- 엔티티를 컨버팅 해주는 기능을 Spring data JPA에서 제공한다.
    - PK 값을 PathVariable로 받을때 바로 엔티티객체로 조회해서 사용이 가능함.

`도메인 클래스 컨버터 사용시 주의점`
- 도메인 클래스 컨버터를 이용한 엔티티는 매우 간단한 케이스에서만 사용이 가능하다.
- 트랜잭션 범위가 없는 케이스에서 조회를했기 때문에 엔티티를 수정하여도 DB에 반영되지 않는다.

> 도메인 클래스 컨버터는 엔티티 조회시 Repository를 사용한다.
