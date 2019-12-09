# JPA 기본편 - 준영속

#### 준영속 상태
- 영속 -> 준영속
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리 (detached)
- 영속성 컨텍스트가 제공하는 기능을 사용하지 못한다.

#### 준영속 상태로 만드는 방법
- em.detach(entity);
    - 특정 엔티티만 준영속 상태로 전환
- em.clear();
    - 영속성 컨텍스트를 완전히 초기화
- em.close();
    - 영속성 컨텍스트를 종료

```java
// 영속 상태
Member findMember2 = entityManager.find(Member.class, 100L);
// 더티 체킹으로 update 쿼리를 날려줌
findMember2.setName("HELLOJPA2");
// 준영속 상태
// 영속성 컨텍스트에서 더이상 관리하지 않는다.
// 트랜잭션이 커밋되더라도 아무일도 일어나지 않는다.
// 직접 사용할 일이 없다.
entityManager.detach(findMember2);

// 영속성 컨텍스트를 비워버린다. (초기화)
// 마찬가지로 아무런 일이 일어나지 않는다.
entityManager.clear();

// 영속성 컨텍스트를 종료한다.
entityManager.close();
```
