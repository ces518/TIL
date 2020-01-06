# JPA 기본편 - 프로젝션

#### 프로젝션
- SELECT 절에 조회할 대상을 지정하는 것이다.
    - 어떤것을 조회할것이라고 지정하는것
- 프로젝션대상: 엔티티, 임베디드 타입, 스칼라타입(숫자, 문자 등 기본 데이터 타입)

```java
select m from Member m; // 엔티티 프로젝션
select m.team from Member m; // 엔티티 프로젝션 , TEAM을 가져온다.
select m.address from Member m; // 임베디드 타입 프로젝션
select m.username, m.age from Member m; // 스칼라타입 프로젝션
// DISTINCT로 중복을 제거 할 수 있다.
```

#### 프로젝션 - 여러 값 조회
1. Query 타입으로 조회
    - Object[] 배열형태로 리턴한다.
2. Object[] 타입으로 조회
    - 제네릭을 사용한다.
3. new 명령어로 조회
    - 단순 값을 DTO로 바로 조회해온다.
    - 패키지명을 포함한 전체 클래스명 입력이 필요하다.
        - QueryDSL 에서는 이런 부분이 해소된다.
    - 순서와 타입이 일치하는 생성자가 필요하다.
    
```java
// 스칼라 타입 프로젝션
List resultList = em.createQuery("select distinct m.username, m.age from Member m")
        .getResultList();

// 1.Query 타입을 사용하면 내부적인 값들은 Object 배열로 리턴된다.
// 2.제네릭을 사용하여 이 과정을 생략할 수 있다.
Object o = resultList.get(0);
Object[] result = (Object[]) o;

// 3.new Operation 사용
// DTO를 활용하여 DTO 타입으로 받아온다.
List<MemberDTO> memberDtos = em.createQuery("select new jpql.MemberDTO(m.username, m.age) from Member m", MemberDTO.class)
        .getResultList();
```
