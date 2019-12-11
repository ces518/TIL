# JPA 기본편 - 기본 키 매핑

#### 기본 키 매핑
- @Id
- @GeneratedValue

#### 기본 키 매핑 방법
- 직접 할당
    - @Id 만 사용한다.
- 자동생성 
    - @GeneratedValue
    - IDENTITY
        - 데이터베이스에 위임, MYSQL
    - SEQUENCE
        - 데이터베이스 시퀀스 사용, ORACLE
        - @SequenceGenerator 사용
    - TABLE
        - 키 생성용 테이블을 사용한다. 모든 DB에서 사용
        - @TableGenerator 필요
    - AUTO
        - 방언에 따라 자동지정, 기본값

> 보통 데이터베이스에 위임해서 사용하는것을 권장한다.

#### IDENTITY 전략 - 특징
- 기본 키 생성을 데이터베이스에 위임한다.
- 주로 MySQL, PostrgreSQL, SQL Server, DB2 에서 사용한다.
- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 을 실행한다.
- AUTO_INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있음
- IDENTITY 전략은 em.persist() 시점에 즉시 INSERT_SQL을 실행하고, DB에서 식별자를 조회한다.

##### 단점
- ID 값을 알 수 있는 시점이 DB에 등록 되어야 할 수 있다.
- 영속성 컨텍스트에서 관리하려면 엔티티의 ID 값이 있어야한다.
- JPA입장에서는 제약조건(persist를 호출하는 시점에 DB에 쿼리를 해야함)이 생긴다.

#### SEQUENCE 전략 - 특징
- 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트 이다.
- ORACLE, PostgreSQL, DB2, H2에서 사용한다.

- persist 하는 시점에 DB Insert SQL 은 날리지 않고, 시퀀스 값만 얻어서 영속성 컨텍스트에 저장한뒤 트랜잭션 커밋시점에 Insert SQL을 날린다.

> @SequenceGenerator를 사용하지 않으면 기본적으로 Hibernate Sequence를 사용한다.

* 각 테이블 별로 Sequence를 관리하고 싶다면, @SequenceGenerator를 사용

##### @SequenceGenerator
- name
    - 식별자 생성기 이름
    - 필수 값
- sequenceName
    - 데이터베이스에 등록되어있는 시퀀스명
    - 기본값은 Hibernate_sequence
- initialValue
    - DDL 생성시에만 사용된다.
    - 처음 시작할 수를 지정한다.
    - 기본값은 1
- allocationSize
    - 시퀀스 한번 호출에 증가하는 수 (성능 최적화에 사용된다.)
    - 데이터베이스 시퀀스 값이 하나씩 증가하도록 되어있따면 반드시 1로 설정해야한다.
    - 기본값은 50
- catalog, schema
    - 데이터베이스, catalog, schema 이름

```java
@SequenceGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    sequenceName = "MEMBER_SEQ",
    initialValue = 1, allocationSize = 1)
```

##### allocateSize
- allocateSize 기본값이 50인 이유
- allocateSize가 1인 경우 매번 등록때마다 sequence.nextVal 로 DB에 매번 호출하게 되어 성능상 이슈가 발생한다.
- 하지만 hiberante의 기본 설정으로 allocateSize가 50 이면, DB에는 미리 50을 올려놓고, 메모리상에서 1씩 올려서 50까지 사용한다.
- 만약 50까지 다사용했다면 다시 호출하여 DB의 값은 100으로 올려 이전 과정을 반복한다.
- 이 방법을 사용하면 다수 WAS를 사용해도 동시성 문제가 해결된다.


#### Table 전략
- 키 생성 테이블을 하나 만들어, 데이터베이스 시퀀스를 흉내내는 전략
- 모든 데이터베이스에 적용이 가능하지만, 성능이 떨어진다.


##### @TableGenerator 와 속성
```java
@TableGenerator(
        name = "MEMBER_SEQ_GENERATOR",
        table = "MY_SEQUENCES",
        pkColumnValue = "MEMBER_SEQ",
        allocationSize = 1)
```

`속성`
- name
    - 식별자 생성기 이름
    - 필수값
- table
    - 키 생성 테이블명
    - hibernate_sequences
- pkColumnName
    - 시퀀스 컬럼명
    - sequence_name
- valueColumnName
    - 시퀀스 값 컬럼명
    - next_val
- pkColumnValue
    - 키로 사용할 값 이름
    - 엔티티명
- initialValue
    - 초기 값, 마지막으로 생성된 값이 기준이다.
    - 기본값은 0
- allocationSize
    - 시퀀스 한번 호출에 증가하는 수 (성능 최적화에 사용된다.)
    - 기본값은 50
- catalog, schema
    - 데이터베이스 catalog, schema 이름
- uniqueConstraint
    - 유니크 제약조건을 지정


#### 권장하는 식별자 전략
- 기본 키 제약조건 : not null, unique, 변하면 안됨
- **미래까지 이 조건을 만족하는 자연키**는 찾기 어렵다.
- 대리키 (대체키) 를 사용하자.
- 비지니스와 관련 없는 값을 키로 사용하는것을 추천한다.
- 주민등록 번호도 기본키로 적절하지 않다.
- 권장
    - Long + 대체키 + 키 생성 전략

> auto increment, sequence, uuid, generator 등을 사용하는것을 권장한다.
