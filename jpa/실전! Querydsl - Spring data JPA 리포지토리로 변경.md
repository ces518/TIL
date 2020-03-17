# 실전! Querydsl - Spring data JPA 리포지토리로 변경

#### Spring data JPA 리포지토리로 변경

`MemberRepository`
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    // 메소드 명으로 쿼리생
    List<Member> findByUsername(String username);
}
```

`MemberRepositoryTest`
```java
@SpringBootTest
@Transactional
class MemberRepositoryTest {

    @Autowired EntityManager em;

    @Autowired MemberRepository memberRepository;

    @Test
    public void basicTest() throws Exception {
        Member member1 = new Member("member1", 10);
        memberRepository.save(member1);
        Optional<Member> memberOptional = memberRepository.findById(member1.getId());
        Member findMember = memberOptional.get();

        assertThat(findMember).isEqualTo(member1);

        List<Member> result1 = memberRepository.findAll();
        assertThat(result1).containsExactly(member1);

        List<Member> result2 = memberRepository.findByUsername("member1");
        assertThat(result2).containsExactly(member1);
    }
}
```

> 기본적인 CRUD는 Spring data JPA 에서 지원해 주는것을 사용
