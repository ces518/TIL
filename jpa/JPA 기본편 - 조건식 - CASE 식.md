# JPA 기본편 - 조건식 - CASE 식

#### 조건식 - CASE 식
- 기본 CASE 식
```java
select
    case 
        when m.age <= 10 then '학생요금'
        when m.age >= 60 then '기본요금'
        else '일반요금'
    end
from Member m;
```

- 단순 CASE 식
```java
select
    case t.name
        when '팀A' then '인센티브110%'
        when '팀B' then '인센티브120%'
        else '인센티브105%'
    end
from Team t;
```

- COALESCE
    - 하나씩 조회해서 null이 아닐경우 반환
```java
// 회원 이름이 null이라면 이름 없는 회원 이라고 출력한다.
em.createQuery("select coalesce(m.username, '이름 없는 회원') from Member m");
```

- NULLIF
    - 두 값이 같으면 Null 반환, 다르면 첫번째 값 반환
```java
// 이름이 관리자 라면 null로 출력
em.createQuery("select nullif(m.username, '관리자') from Member m");
```

> 위 조건식들은 모두 SQL 표준이기 때문에 JPA에서 제공하는 어떠한 DB에서도 모두 지원한다.
