# JPA 기본편 - 다형성 쿼리

#### JPQL 다형성 쿼리
- Item
    - id
    - name
    - price
- Album extends Item
    - artist
- Movie extends Item
    - director
    - actor
- Book extends Item
    - author
    - isbn

위와 같은 구조의 상속 구조의 모델일 경우 JPA에서 특수한 기능을 지원한다.

#### TYPE
- 조회 대상을 특정 자식으로 한정할 수 있다.
    - Item 중 Book, Movie를 조회

```java
select i from Item i
where type(i) IN (Book, Movie);
```
```sql
select i.* from Item i
where i.DTYPE in ('B', 'M');
```

> TYPE 이 DTYPE으로 변환되어 SQL로 나가게 된다. DTYPE 은 구분컬럼으로 바뀌어서 SQL로 나가게 된다.

#### TREAT ( JPA2.1 )
- 자바의 타입 캐스팅과 유사하다.
- 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰때 사용한다.
- from, where, select(하이버네이트 지원) 에서 사용 가능하다.

- 부모인 Item과 자식인 Book이 있다.
```java
select i from Item i
where treat(i as Book).author = 'kim'
```
```sql
select i.* from Item i
where i.DTYPE = 'B' and i.author = 'kim';
```

> 다음과 같이 부모의 타입을 자식타입으로 다운 캐스팅하여 조회가 가능하다.
