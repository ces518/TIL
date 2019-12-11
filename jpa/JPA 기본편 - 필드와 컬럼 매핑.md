# JPA 기본편 - 필드와 컬럼 매핑

#### 요구사항
- 1. 회원은 일반회원과, 관리자로 구분해야 한다.
- 2. 회원가입일, 수정일이 있어야한다.
- 3. 회원을 설명할 필드가 있어야한다. 길이제한이 없다.


#### Member
```java
@Entity
// USER 라는 테이블에 매핑을 해준다.
@Table(name = "USER")
public class Member {

    @Id
    private Long id;

    // DB에는 name으로 사용한다.
    @Column(name = "name")
    private String name;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifedDate;

    @Lob
    private String description;
}
```

#### 매핑 어노테이션 정리
- @Column
    - 컬럼과 매핑한다.
- @Temporal
    - 날짜 타입과 매핑한다.
- @Enumerated
    - enum 타입과 매핑한다.
- @Lob
    - BLOB, CLOB 타입 과 매핑한다.
- @Transient
    - 특정 필드를 컬럼에서 제외하고 싶을때 사용한다.


#### @Column 의 속성
- name
    - 필드와 매핑할 테이블 컬럼의 이름
    - 기본값은 객체의 필드 명이다.
- insertable, updatable
    - 등록, 변경 가능여부
    - 값이 변경되었을때, insert, update를 DB에 반영할 것인지 여부
    - 기본값은 TRUE
- nullable (DDL)
    - null 허용 여부를 설정한다.
    - false 설정시 not null 제약 조건이 붙는다.
    - 기본값은 TRUE
- unique (DDL)
    - @Table의 uniqueConstraints를 많이 사용한다.
    - unique 제약 조건을 걸어준다.
    - 잘 사용하지 않는다.
    - 제약 조건 명이 알아보기 힘들다.
- columnDefinition
    - 데이터베이스 컬럼 정보를 직접 줄 수 있다.
    - 특정 DB에 종속적인 조건을 줄 수 있다.
- length    
    - 컬럼의 길이를 정의한다.
    - String 일 경우에만 사용한다.
    - 기본값 255
- precision, scale (DDL)
    - 큰 숫자나 소수점을 사용할때 이용한다.

#### Enum 타입 사용시 주의사항
- @Enumerated 의 기본 값은 ORDINAL 이다.
- ORDINAL
    - enum의 순서를 저장
- STRING
    - enum의 이름을 저장

> 순서를 저장하기 때문에 도중에 새로운 enum이 추가되거나, 순서가 바뀔경우 버그 발생..

#### @Temporal의 속성
- value
    - TemporalType.DATE
        - 날짜 타입
    - TemporalType.TIME
        - 시간 타입
    - TemporalType.TIMESTAMP
        - 날짜 + 시간 타입

> 참고: LocalDate, LocalDateTime 을 사용할 경우 생략 가능하다 (하이버네이트 최신버전 에서 지원)

#### @Lob
- 지정할 수 있는 속성이 없다.
- 매핑하는 필드가 String 이라면, Clob, 나머지는 Blob으로 매핑된다.

#### @Transient
- 필드매핑 을 하지않고, 데이터베이스 저장, 조회를 하지않을때 사용함
- 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을때 사용한다.
