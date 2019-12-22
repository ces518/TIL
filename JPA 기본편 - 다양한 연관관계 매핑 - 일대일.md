# JPA 기본편 - 다양한 연관관계 매핑 - 일대일

#### 일대일 관계
- 반대도 일대일 관계이다
- 주 테이블이나 대상 테이블 중 외래키 선택 가능
    - 주테이블에 외래키
    - 대상테이블에 외래키
- 외래 키에 데이터베이스 유니크 제약조건 추가

#### 주 테이블에 외래키 단방향
- Member
    - id
    - Locker locker
    - username
- Locker
    - id
    - name
- MEMBER
    - MEMBER_ID (PK)
    - LOCKER_ID (FK, UNI)
    - USERNAME
- LOCKER
    - LOCKER_ID (PK)
    - NAME


#### 주 테이블에 외래키 단방향 정리
- 다대일 양방향 매핑과 유사하다.

#### 주 테이블에 외래키 양방향
- Member
    - id
    - Locker locker
    - username
- Locker
    - id
    - name
    - Membmer memeber (읽기전용)
- MEMBER
    - MEMBER_ID (PK)
    - LOCKER_ID (FK, UNI)
    - USERNAME
- LOCKER
    - LOCKER_ID (PK)
    - NAME


#### 주 테이블에 외래키 양방향 정리
- 다대일 양방향 매핑처럼 외래키가 있는곳이 연관관계의 주인
- 반대편은 mappedBy 적용


#### 대상 테이블에 외래키 단방향
- Member
    - id
    - Locker locker
    - username
- Locker
    - id
    - name
- MEMBER
    - MEMBER_ID (PK)
    - USERNAME
- LOCKER
    - LOCKER_ID (PK)
    - MEMBER_ID (FK, UNI)
    - NAME

> 대상 테이블에 외래키 단방향 매핑은 ? 지원하지 않는 매핑..

#### 대상 테이블에 외래키 단방향 정리
- 단뱡항 관계 JPA 지원하지 않는다.
- 양방향은 지원


#### 대상 테이블에 외래키 양방향
- Member
    - id
    - Locker locker
    - username
- Locker
    - id
    - name
    - Member member
- MEMBER
    - MEMBER_ID (PK)
    - USERNAME
- LOCKER
    - LOCKER_ID (PK)
    - MEMBER_ID (FK, UNI)
    - NAME

> 사실 말에 어폐가 있는것이지 Member를 주인으로 관리하는 것이다.

#### 대상 테이블에 외래키 양방향
- 사실 일대일 주 테이블에 외래키 양방향과 매핑 방법은 같다.


#### 일대일 정리
- 주 테이블에 외래키
    - 주 객체가 대상 객체의 참조를 가지는 것 처럼
    - 주 테이블에 외래키를 두고 대상 체이블을 찾음
    - 객체지향 개발자 선호
    - JPA 매핑 편리
    - 장점
        - 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
    - 단점
        - 값이 없다면 외래키에 null 허용
- 대상 테이블에 외래키
    - 대상 테이블에 외래 키가 존재
    - 전통적인 데이터베이스 개발자 선호
    - 장점
        - 주 테이블과 대상 테이블을 일대일엥서 일대다 관계로 변경할때 테이블 구조 유지
    - 단점
        - 프록시 기능의 한계로 지연로딩으로 설정해도 항상 즉시 로딩된다.
        - 쿼리가 무조건 나가기 때문에 ...
