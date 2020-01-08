# JPA 기본편 - JPQL 기본 함수

#### JPQL 기본 함수
- CONCAT
- SUBSTRING
- TRIM
- LOWER, UPPER
- LENGTH
- LOCATE
- ABS, SQRT, MOD
- SIZE, INDEX(JPA 용도)

```java
// 기본 함수
em.createQuery("select concat('a', 'b') from Member m");
em.createQuery("select substring(m.username, 2, 3) from Member m");
em.createQuery("select trim(m.username) from Member m");
em.createQuery("select lower(m.username) from Member m ");
em.createQuery("select upper(m.username) from Member m ");
em.createQuery("select length(m.username) from Member m");
// 해당 문자열의 위치를 반환
em.createQuery("select locate('de', 'abcdef') from Member m ");
em.createQuery("select abs(1.0) from Member m");
// 연관관계를 맺고 있는 컬렉션의 크기를 알려준다. (JPA 함수)
em.createQuery("select size(t.members) from Team t");
// 값 타입 컬렉션에서의 인덱스를 반환한다. 사용하지 않는것을 추천 (JPA함수)
em.createQuery("select index(t.members) from Team t");
```

#### 사용자 정의 함수
- 애플리케이션을 개발하다보면 DB에 존재하는 함수를 사용할 떄가 있다.
    - JPQL에서는 이를 알 방법이 없다.
- 하이버네이트는 사용전 방언에 추가해야한다.
- 상속하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다.

> 다행히도 각 벤더별로 Dialect 에서 DB 종속적인 함수들이 등록 되어있다.

`사용자 정의 함수 등록`
- 사용자 정의 함수를 등록하려면, Dialect를 생성해야하고, 기존에 사용하던 Dialect를 상속받은뒤 생성자에서 등록을 해주면 된다
```java
public class MyH2Dialect extends H2Dialect {
    public MyH2Dialect() {
        // 사용자 정의 함수를 사용하려면, Dialect를 생성한뒤, 생성자에서 등록을 해주어야한다.
        registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
    }
}
```

`사용자 정의 함수 사용`
```java
// 사용자 정의 함수
// 이름1, 이름2, 이름3 형태로 한줄로 합쳐버린다.
em.createQuery("select function('group_concat', m.username) from Member m");
// 하이버네이트 일경우
em.createQuery("select group_concat(m.username) from Member m");
```
