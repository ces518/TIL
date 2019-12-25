# JPA 기본편 - 고급매핑 - 상속관계 매핑

#### 상속관계 매핑
- 객체는 상속 관계가 있다.
- 관계형 데이터베이스는 상속 관계가 없다.
    - 비스무리하게 지원하는 데이터 베이스도 있지만 객체의 상속관계와 다르다.
- 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 그나마 유사하다
- 상속관계 매핑
    - 객체의 상속 구조와, DB의 슈퍼타입 서브 타입 관계를 매핑한다.

- 슈퍼타입 서브타입 을 매핑하는 방법은 3가지 방법이 있다.
- 1. 조인 전략
    - ITEM
        - ALBUM
        - MOVIE
        - BOOK
    - 공통 속성을 ITEM으로 만들고, 조회해올때 JOIN해서 가져온다. INSERT는 2번한다.

- 2. 단일 테이블 전략
    - 논리 모델을 한테이블로 모두 합쳐버리는 방법

- 3. 구현 클래스마다 테이블 생성
    - 클래스 마다 각 테이블을 모두 가지고 있는 방법

> 객체 입장에서는 모두 같다.


#### 주요 애노테이션
- @Inheritance
    - 상속관계 테이블 매핑 애노테이션
    - strategy
        - SINGLE_TABLE: 단일 테이블 형태 모델 (기본값)
            - DTYPE이 필수로 생성
        - JOIN_TABLE: 조인 테이블 형태 모델
        - TABLE_PER_CLASS: 구현 클래스별 테이블을 각각 생성하는 모델
- @DiscriminatorColumn
    - DTYPE 컬럼(명)을 지정하는 애노테이션
    - 슈퍼클래스에 존재해야함
- @DiscriminatorValue("A")
    - DTYPE 컬럼에 들어가야하는 값을 지정하는 애노테이션
    - 서브클래스에 존재해야함

> 단일 테이블전략을 사용하는경우 DTYPE이 존재하는것이 운영상 좋음

> 재미있는 점은 각 전략 3가지를 스위칭하는 과정에서 소스코드는 거의 손댄것이 없음 애노테이션을 변경해주는것만으로 전략 변경이 가능해짐
- 조인 전략을 사용하다가 단일 테이블 전략으로 변경할경우 JPA가 아니라면 소스코드를 상당히 많이 뜯어 고쳐야 한다.
- JPA의 장점

#### 조인전략
- 장점
    - 데이터가 정규화되서 들어간다.
    - 외래 키 참조 무결성 제약조건을 활용 가능하다.
    - 저장공간 활용 효율화

- 단점
    - 조회시 조인을 많이 사용한다. 성능문제
    - 조회 쿼리가 복잡하다.
    - 데이터 저장시 INSERT SQL이 2번 실행된다.
- 조인이나 저장 SQL은 사실 상 큰 문제는 아님

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED) // 기본 전략은 단일테이블 전략
// JOINED 는 조인테이블 전략 모델로 생성
@DiscriminatorColumn // DTYPE
public abstract class Item {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private int price;
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item {
    private String artist;
}
```

> 기본적으론 조인 전략이 정석이라고 생각하면 된다.
- 객체입장과 설계입장에서도 깔끔하게 나온다.

#### 단일테이블 전략
- 장점
    - 조인이 필요없다. 조회성능이 빠름
    - 조회쿼리가 단순하다.

- 단점 
    - 자식엔티티가 매핑한 컬럼은 모두 NULL을 허용해야 한다.
    - 데이터 무결성에 있어서 문제가 있음
    - 단일 테이블에 모든것을 저장함 따라서 조회성능이 오히려 느려질 수 있다.

```java
@Entity
@Inheritance
@DiscriminatorColumn
public abstract class Item {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private int price;
}
```


#### 구현클래스 마다 테이블 전략
- 실무에서 사용해선 안되는 전략
- 데이터베이스 설계자, ORM 전문가 모두 추천하지 않음

- 장점
    - 서브타입을 명확하게 구분해서 처리해서 효과적
    - NOT NULL 제약조건을 사용 가능

- 단점
    - 여러 자식 테이블을 함께 조회할떄 성능이 느리다.
    - 자식 테이블을 통합해서 쿼리하기 어렵다.

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private int price;
}
```

#### 정리
- 상속관계 매핑의 정석은 JOINED 전략이라고 생각 할 것
- 구현클래스마다 테이블 전략은 절대 사용하지 말 것
- 설계및 개발시 적절한 트레이드오프를 통해 전략을 사용할것
