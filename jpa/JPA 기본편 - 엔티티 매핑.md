# JPA 기본편 - 엔티티 매핑

#### 엔티티 매핑
- 객체와 테이블 매핑
    - @Entity, @Table
- 필드와 컬럼 매핑
    - @Column
- 기본 키 매핑
    - @Id
- 연관관계 매핑
    - @ManyToOne, @JoinColumn

#### 객체와 테이블 매핑

##### @Entity
- @Entity가 붙은 클래스는 JPA가 관리하며 엔티티라고 한다.
- JPA를 사용하여 테이블과 매핑할 클래스라면 필수이다.
- 주의 할점
    - **기본생성자 필수**
    - final 클래스, enum, interface, inner 클래스 사용 X
    - DB에 저장할 필드에는 final 사용 X

`속성`
- name 속성
    - JPA에서 사용할 엔티티 이름을 지정한다.
    - 기본값: 클래스 이름을 그대로 사용
    - 같은 클래스 명이 없다면 가급적 기본값을 사용한다.

##### @Table
- @Table은 엔티티와 매핑할 테이블 지정

`속성`
- name
    - 매핑할 테이블 명
    - 기본값은 엔티티 명을 사용한다.
- catalog
    - 데이터베이스 catalog 매핑
- schema
    - 데이터베이스 schema 매핑
- uniqueConstrains
    - DDL 생성시 유니크 제역조건 생성
