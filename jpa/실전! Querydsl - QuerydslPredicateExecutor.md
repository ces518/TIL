# 실전! Querydsl - QuerydslPredicateExecutor

#### Sprign data JPA가 제공하는 Querydsl 기능
- 제약이 커서 실무에서 사용하기에는 많이 부족하다.
- 간단히 소개하고, 왜 부족한지 알아본다.

#### 인터페이스 지원 - QuerydslPredicateExecutor
- https://docs.spring.io/spring-data/jpa/docs/2.2.5.RELEASE/reference/html/#reference
- 파라메터를 Querydsl Predicate 를 지원한다.
- Where 문에 넣는 조건들을 편하게 사용 가능하게 제공한다.

```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom, QuerydslPredicateExecutor<Member> {
    // 메소드 명으로 쿼리생
    List<Member> findByUsername(String username);
}
```
- QuerydslPredicateExecutor 인터페이스를 상속받으면 바로 사용이 가능하다.

```java
@Test
public void querydslPredicateExecutorTest() throws Exception {
// given
Team teamA = new Team("teamA");
Team teamB = new Team("teamB");
em.persist(teamA);
em.persist(teamB);

Member member1 = new Member("member1", 10, teamA);
Member member2 = new Member("member2", 20, teamA);

Member member3 = new Member("member3", 30, teamB);
Member member4 = new Member("member4", 40, teamB);

em.persist(member1);
em.persist(member2);
em.persist(member3);
em.persist(member4);


// when
Iterable<Member> result = memberRepository.findAll(
        QMember.member.age.between(20, 40)
                .and(QMember.member.username.eq("member1")));
for (Member member : result) {
        System.out.println("member = " + member);
}

// then
}
```
- findAll() 과 같은 메소드의 파라메터에서 QType 클래스를 바로 사용해서 조건절을 생성할 수 있다.


`한계점`
- 조인 X (묵시적 조인은 가능하지만 left join이 불가능하다.)
- 클라이언트가 Querydsl에 의존해야 한다. 서비스 클래스가 Querydsl이라는 구현 기술에 의존해야 한다.
- 복잡한 실무 환경에서 사용하기에는 한계가 명확하다.
  - 테이블 하나로 돌아가는 작은 경우에만 사용이 가능함

> Pageable, Sort를 모두 지원하고 정상 동작한다.
