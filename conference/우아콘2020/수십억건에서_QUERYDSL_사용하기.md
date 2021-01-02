# 수십억건에서 QueryDsl 사용하기
- 천만건에서 10억건까지 늘어나면서 QueryDsl JPA 개선 TIP

## 테스트 환경
- OpenJDK 1.8
- Querydsl-JPA 4.2.1
- MySQL 5.6
    - 인덱스 컨디션 푸시다운, 서브쿼리 최적화 기능 제공
    
### 워밍업
- 상속과 구현 사용하지 않기
- XXCustom / Impl 의 쌍을 사용하지 않고 사용하고 싶다.
    - JPAQueryFactory 만 있다면 사용이 가능하다.
- 동적쿼리
    - 보통 BooleanBuilder 를 사용한다.
    - BooleanExpression 을 사용하자.
    

### 성능 개선 - Select
`Querydsl 의 exist 사용 금지`
- 2500만건기준
- exist 는 첫번째 데이터를 발견하는 순간 끝난다.
- count 쿼리 는 모든 데이터를 조회한다.
- querydsl 의 exist 는 count() > 0 을 사용한다.

`해법 - limit 1로 조회 제한해서 찾는다.`
```java
Integer fetchOne = queryFactory
        .selectOne()
        .from(book)
        .where(book.id.eq(bookId))
        .fecthFirst();

return fetchOne != null;
```
> 주의할 점은 null 이냐 아니냐로 체크를 해야한다. 결과가 없다면 null 이 반환되기 때문.

`Cross Join 회피`
- 성능이 좋지 않다.
- 일부 디비는 최적화를 해주지만 피하는것이 좋다.
- QueryDSL 사용시 묵시적 조인을 사용하게되면 Cross Join 이 발생한다.
- Spring data JPA 를 사용하더라도 동일한 문제가 발생함!!

> 명시적 조인을 사용하자.

`Entity 보다는 Dto를 우선`
- 무조건 Entity 를 사용해야한다고 오해를 한다.
- 엔티티 조회시 
  - N + 1 쿼리등 문제가 발생
    - 단순조회 기능에서 성능이슈 요소가 많다.
- Dto 조회
    - 고강도 성능개선 or 대량 데이터 조회 필요시

`조회 컬럼 최소화 하기`
- 이미 알고있는 값을 조회할 필요는 없다.
```java
int bookNo = 1;
queryFactory
        .select(
                Projections.fields(BookPageDto.class,        
                book.name,
                Expressions.asNumber(bookNo).as("bookNo"),
                book.id
        ))
        .from(book)
        .where(book.bookNo.eq(bookNo))
        .offset(1)
        .limit(10)
        .fetch();
```
> as 를 사용해서 조회대상에서 제외시킨다.

`Select 컬럼에 Entity 자제 - 불필요한 컬럼 조회`
- 엔티티의 모든 컬럼이 조회된다.
- @OneToOne 관계시 N + 1 문제 발생 -> OneToOne 은 Lazy 로딩이 안된다.

`Select 컬럼에 Entity 자제 - distinct`
- distinct 를 위한 임시 테이블을 생성하기 위한 비용 절감을 위함


`Group By 최적`
- MySQL 에서 Group By 실행시 FileSort 가 필수적으로 발생함
- MySQL 에서 order by null 사용시 FileSort 가 제거됨
- querydsl 에선 지원하지 않음 우리가 직접 구현해야한다.

`OrderByNull 클래스를 직접 구현`
```java
public class OrderByNull extends OrderSpecifier {
    public static final OrderByNull DEFAULT = new OrderByNull();
    
    private OrderByNull() {
        super(Order.ASC, NullExpresson.DEFAULT, Default);
    }
}
```
> 2500 만건 기준 성능차이가 심함.
> 조회 결과가 100건 이하라면 애플리케이션에서 정렬할것 을 추천한다.
> 페이징의 경우 order by null 사용하지 못한다.

`커버링 인덱스`
- 쿼리를 충족시키는데 필요한 모든 컬럼을 갖고 있는 인덱스
- NoOffset 방식과 더불어 페이징 조회성능을 향상시키는 보편적인 방법
- JPQL 은 from 절 서브쿼리 지원하지 않음
- 커버링 인덱스 조회는 나눠서 조회한다.
- Cluster Key (PK) 를 커버링 인ㄷ게스로 빠르게 조회한뒤
- 조회된 Key 로 SELECT 컬럼들을 후속 조회한다.

### 성능개선 Update/Insert

`일괄 Update 최적화`
- JPA 사용시 더티체킹을 많이 사용한다.
- 대량의 데이터를 수정할때 더티체킹을 사용하면 성능상 많이 손해를 본다.
- QueryDSl 을 사용해서 한방 쿼리 사용할것.
> 1만건 단일 컬럼기준 2천배 이상 차이남.
> 하이버네이트 캐시는 일괄 업데이트시 캐시 갱신이 필요하다.

- 더티체킹
  - 실시간 비즈니스 처리, 실시간 단건 처리
- Querydsl.Update
  - 대량의 데이터 일괄처리, 배치 등

> 엔티티가 필요한게 아니라면 Querydsl / Dto 를 통해 딱 필요한 항목만 사용하라..

`BulkInsert`
- JPA BulkInset 는 자제할것
- Jdbc 는 rewriteBatchedStatements 로 insert 합치기 옵션이 있다.
- JPA 는 auto_increment 일때 insert 합치기가 적용되지 않음..

### Querydsl != Querydsl-JPA
- Querydsl 은 하위 모듈이 존재함.
  - Querydsl-JPA -> JPQL
  - Querydsl-SQL -> NativeSQL
  - Querydsl-MongoDB -> MongoQuery
  - Querydsl-ElasticSearch -> ESQuery

- Querydsl-SQL 은 테이블 스캔 기반 QClass 를 생성...
- EntityQL 을 사용하면 애노테이션 기반으로 QClass 를 생성한다.
- 제약조건
  - Gradle 5 이상
  - 애노테이션에 @Column(name) 이 필수
  - 기본형 타입 사용 불가능함 - 모든 컬럼은 래퍼클래스여야한다.
  - @Embedded 미지원
  
## 정리
- 상황에 따라 ORM / Query 방식 골라 사용
- JPA / RealMySQL 필독..