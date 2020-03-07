# 실전! 스프링 데이터 JPA - JPA Hint & Lock

#### JPA Hint 
- JPA 쿼리 힌트 (SQL 힌트가 아닌 JPA 구현체(Hibernate)에게 제공하는 힌트)

`쿼리 힌트 사용`
```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
Member findReadOnlyByUsername(@Param("name") String username);


@Test
public void queryHint() {
    // given

    // readOnly Hint를 제공한다.
    Member savedMember = memberRepository.save(new Member("member1", 10));
    Member savedMembers = memberRepository.save(new Member("members", 10));
    em.flush();
    em.clear();

    // when

    Member findMember = memberRepository.findById(savedMember.getId()).get();
    // 기존 동작이라면 영속성 컨텍스트에서 관리하는 엔티티이기 때문에 변경감지가 동작하여 update 쿼리가 날아간다.
    // 변경 감지는 내부적으로는 최적화를 하겠지만 결국 두개의 객체를 가지고 있기 때문에 메모리를 더 잡아먹는다.
    findMember.setUsername("member2");

    // 100% 조회로만 사용할것이라면 최적화하는 방식이 존재한다.
    // JPA에서는 제공하지 않고, Hibernate에서만 제공하는 기능
    // 내부적으로 snapshot을 사용하지 않기때문에 변경감지가 일어나지 않는다.
    Member findMember2 = memberRepository.findReadOnlyByUsername("members");
    findMember2.setUsername("members2");

    // 이러한 쿼리힌트를 사용하는것은 극소수이다.
    // 적용하기 전에 성능테스트를 먼저 진행해보고 결정할것.
    // 무조건 다 넣는다고 좋은것은 아님.
    // 정말 성능 최적화가 필요하다면 이미 캐시를 사용해야하기 떄문에 Redis등이 존재할것임

    // then
}
```

- 기존 동작대로라면 영속성 컨텍스트에서 관리하는 엔티티 이기때문에 setUsername 메소드를 사용하면 변경 감지가 동작하여 update쿼리가 날아간다.

> 변경 감지는 내부적으로 최적화를 하더라도 결국 스냅샷이 필요하기 때문에 메모리를 더 잡아먹게 된다.

- 만약 100% 조회로만 사용할 경우 최적화하는 방식을 제공한다.
    - QueryHint 
    - JPA에서는 제공하지않는 기능이며, Hibernate에서만 제공
- QueryHint를 ReadOnly로 주게되면 내부적으로 스냅샷을 사용하지 않음.
    - 변경감지가 일어나지 않는다.

> 이러한 QueryHint를 사용하는 경우는 매우 극소수이다. 무조건 넣는다고 해서 좋은것은 아니다. 성능 테스트후 판단하여 적용해야 한다.

#### JPA의 Lock
- JPA에서 제공하는 Lock 기능

```java
// select from update
// DB에서 셀렉 할때 Lock을 거는 방식을 JPA에서도 지원한다.
@Lock(LockModeType.PESSIMISTIC_WRITE)
Member findLockByUsername(@Param("name") String username);

@Test
public void lock() {
    // given

    Member savedMember = memberRepository.save(new Member("member1", 10));
    Member savedMembers = memberRepository.save(new Member("members", 10));
    em.flush();
    em.clear();

    // when

    // select 쿼리 후미에 for update가 사용됨
    /*
        select
            member0_.member_id as member_i1_0_,
            member0_.age as age2_0_,
            member0_.team_id as team_id4_0_,
            member0_.username as username3_0_
        from
            member member0_
        where
            member0_.username=? for update
        */
    Member member1 = memberRepository.findLockByUsername("member1");
}
```

- 데이터베이스의 select for update 와 같은 lock 기능을 제공한다.
