# JPA 기본편 - 조인

#### 조인
- 내부 조인
    - select m from Member m join m.team t;

- 외부 조인
    - select m from Member m left join m.team t;

- 세타 조인
    - select count(m) from Member m, Team t where m.username = t.name;
    - 연관관계가 없는것을 조인하는 것
    - 카티시안 곱으로 결과가 나옴 -> 그후 username, name 이 같은애를 추린다.

> 엔티티를 중심으로 SQL 문법이 나간다.

#### 조인 - ON 절
- ON절을 활용한 조인 (JPA 2.1 부터 지원)
- 1.조인대상 필터링
- 2.연관관계 없는 엔티티 외부 조인 (하이버네이트 5.1 부터 가능)


#### 조인 대상 필터링
- 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인
```java
select m, t from Member m left join m.team t on t.name = 'A';
```

#### 연관관계가 없는 엔티티 외부 조인
- 회원의 이름과 팀의 이름이 같은 대상 외부 조인
- Spring Boot 1.5 낮은 버전에서는 잘 안 될수 있음
```java
select m. t from
Member m left join Team t on m.username = t.name
```
