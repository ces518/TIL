# 실전! 스프링 데이터 JPA - 벌크성 수정쿼리

#### 벌크성 수정 쿼리
- JPA는 보통 diry checking을 이용해서 업데이트를 권장한다.
- 하지만 이 방식은 한건 한건씩 진행하는 방식이다.
- 만약 모든 직원의 연봉을 10%씩 인상해야한다면 ?
    - 하나하나 진행하기엔 매우 비효율적
    - 이를 해결하기위해 벌크성 수정쿼리 기능을 제공한다.

#### 순수 JPA를 사용한 벌크연산
```java
// 파라메터로 넘어온 나이보다 큰 멤버의 나이를 1씩 증가
public int bulkAgePlus(int age) {
    return em.createQuery("update Member m set m.age = m.age + 1 where m.age >= :age")
            .setParameter("age", age)
            .executeUpdate();
}
```

```java
// TEST 코드
@Test
public void bulkUpdate() {
    // given
    memberJpaRepository.save(new Member("member1", 10));
    memberJpaRepository.save(new Member("member2", 19));
    memberJpaRepository.save(new Member("member3", 20));
    memberJpaRepository.save(new Member("member4", 21));
    memberJpaRepository.save(new Member("member5", 30));
    memberJpaRepository.save(new Member("member6", 17));
    memberJpaRepository.save(new Member("member7", 40));

    // when
    int resultCount = memberJpaRepository.bulkAgePlus(20);

    // then
    assertThat(resultCount).isEqualTo(4);
}
```

- JPQL을 사용하여 벌크연산을 진행한다.
- 우리가 기대한 대로 4개의 데이터가 수정된다.


#### Spring data JPA를 사용한 벌크연산
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    // @Modifying 애노테이션이 있어야 update쿼리를 실행한다.
    // 데이터 변경이 일어나는 쿼리는 @Modifying 애노테이션을 사용할것.
    // 만약 @Modifying애노테이션이 없다면 예외가 발생한다.
    @Modifying(clearAutomatically = true) // 영속성 컨텍스트를 자동으로 비워주는 옵션
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);

}
```

- @Query 애노테이션을 사용하여 JPA와 마찬가지로 JPQL을 작성한다.
- 단 JPA와의 차이점은 @Modifying을 사용해 주어야한다는 점이다.
- 데이터 변경이 일어나는 쿼리는 @Modifying을 사용해 주어야한다.
- 만약 @Modifying애노테이션이 없다면 예외가 발생한다.

```java
// TEST 코드
@Test
public void bulkUpdate() {
    // given
    memberRepository.save(new Member("member1", 10));
    memberRepository.save(new Member("member2", 19));
    memberRepository.save(new Member("member3", 20));
    memberRepository.save(new Member("member4", 21));
    memberRepository.save(new Member("member5", 30));
    memberRepository.save(new Member("member6", 17));
    memberRepository.save(new Member("member7", 40));

    // when
    int resultCount = memberRepository.bulkAgePlus(20);
    // Spring data JPA를 사용한다면 @Modifying의 옵션으로 영속성 컨텍스트 를 비워주는 로직 생략이 가능
    em.flush(); // 변경되지 않은 부분 DB에 반영
    em.clear(); // 영속성 컨텍스트를 비워줌
    // JPA 기본 동작: JPQL을 사용하면 DB에 flush를 한번 해줌

    // JPA 를 사용하면 bulk 연산에서 조심해야 할점이다.
    // 벌크 연산은 영속성 컨텍스트가 아닌 DB에 바로 반영해버리기 때문에 영속성 컨텍스트에 존재하는 엔티티와는 다르다.
    // 벌크 연산 이후에는 영속성 컨텍스트를 날려버려야 한다.
    List<Member> result = memberRepository.findByUsername("member5");
    Member member5 = result.get(0);
    System.out.println("member5.getAge() = " + member5.getAge());
    // then
    assertThat(resultCount).isEqualTo(4);
}
```

- 기본적으로 JPA와 동일하게 동작한다.
- 이전과 마찬가지로 우리가 기대했던 데이터가 수정된다.

`벌크 연산시 주의점`
- Spring data JPA, JPA 모두 마찬가지로 벌크 연산을 사용한다면 주의해야할 점이 있다.
- **벌크 연산은 영속성 컨텍스트가 아닌 DB에 바로 반영 해버리기 때문에 영속성 컨텍스트에 존재하는 엔티티와는 싱크가 맞지 않다.**
- 만약 벌크연산 이후에 수행해야할 비즈니스 로직이 있다면 **벌크 연산 이후에 영속성 컨텍스트를 비워주어야 한다.**

> JPA를 사용한다면 em.flush(), em.clear()를 사용해 비워주고, Spring data JPA를 사용한다면 @Modifying의 옵션으로 영속성 컨텍스트를 비워주도록 하자.
