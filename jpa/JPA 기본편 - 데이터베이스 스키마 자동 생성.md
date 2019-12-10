# JPA 기본편 - 데이터베이스 스키마 자동 생성

#### 데이터베이스 스키마 자동생성
- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 생성된 DDL은 개발 장비에서만 사용
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 뒤 사용

`속성`
hibernate.hbm2ddl.auto
- create
    - 기본 테이블 삭제후 새롭게 생성한다.
- create-drop
    - create와 같으나 종료 시점에 테이블을 삭제한다.
- update
    - 변경된 사항만 추가된다.
    - 컬럼이 지워지는건 되지 않음
- validate
    - 엔티티와 테이블이 정상 매핑 되었는지 확인한다.
- none
    - 사용하지 않음

##### 주의 할점
- 개발 초기단계 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징 과 운영서버는 validate 또는 none

#### DDL 생성 기능
- 제약조건 추가: 회원명은 필수, 널 허용 X
```java
@Column(name = "username", unique = true, length = 10)
String name;
```

> DDL 생성 기능은 DDL을 자동생성할 때만 사용되고, JPA 실행 로직에는 영향을 주지 않는다.
