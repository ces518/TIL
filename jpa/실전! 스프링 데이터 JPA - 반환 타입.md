# 실전! 스프링 데이터 JPA - 반환 타입

#### 반환 타입
- Spring data JPA 는 유연한 반환 타입을 지원한다.
    - optional 지원
```java
List<Member> findListByUsername(String username); // 컬렉션

Member findOneByUsername(String username); // 단건

Optional<Member> findOptionalByUsername(String username); // optional
```

```java
@Test
public void returnType() {
    Member member1 = new Member("AAA", 10);
    Member member2 = new Member("BBB", 20);
    memberRepository.save(member1);
    memberRepository.save(member2);

    /*
        컬렉션
        -> 컬렉션 결과가 없더라도 빈 컬렉션을 반환해준다.
    * */
    List<Member> members = memberRepository.findListByUsername("AAA");

    /*
        단건
        -> JPA에서는 단건 조회 결과가 없을 경우 NoResultException 예외가 발생한다.
        Spring data JPA 에서는 예외없이 null을 반환한다.
        JAVA 8 부터는 Optional을 사용
    * */
    Member member = memberRepository.findOneByUsername("BBB");

    /*
        단건 조회인데 둘이상일 경우 (Optional 상관 없이)
        -> IncorretResultSizeDateAccessException 이 발생 (Spring 예외)
        원래는 NonUniqueResultException 이 발생함 (기존 예외)
        Spring data JPA가 IncorretResultSizeDateAccessException 로 변환해 주는것
        * 예외를 공통된 예외로 변환해주어 데이터베이스 등이 변경되어도 동일한 예외 처리가 가능하게끔 한다.
    */
    Optional<Member> optionalMember = memberRepository.findOptionalByUsername("BBB");
}
```

`컬렉션 타입`
- 조회 결과가 없더라도 null이 아닌 빈 컬렉션을 반환해 준다.

`단건`
- JPA에서는 단건 조회 결과가 없을 경우 NoResultException 예외가 발생한다.
- Spring data JPA 에서는 예외가 발생하지 않고 null을 리턴해준다.

> Java 8 부터는 Optional을 사용할것

#### 단건 조회인데 결과가 둘 이상일 경우
- IncorretResultSizeDataAccessException 이 발생한다. (Spring의 예외)
- 원래는 NonUniqueResultException 이 발생한다. (JPA 예외)
- Spring data JPA가 IncorretResultSizeDataAccessException으로 변환해 준다.
- 공통된 예외로 변환해주어 데이터 엑세스 기술등이 변경되어도 동일한 예외 처리를 할 수 있도록 추상화
