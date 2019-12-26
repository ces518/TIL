# JPA 기본편 - 고급매핑 - @MappedSuperClass

#### @MappedSuperClass
- 상속관계 매핑과는 관련이 없다.
- 공통적인 속성을 매핑할때 사용한다.
- 객체의 입장에서 공통 매핑정보가 필요할때 사용한다.
- 엔티티가아니다, 테이블과 매핑되지 않는다.
- 조회, 검색 불가
- 직접 생성해서 사용할 일이 없으므로 추상클래스 권장

- BaseEntity
    - id
    - name
- Member extends BaseEntity
    - email

> 객체의 입장에서 공통 매핑정보를 상속받아 사용할때 사용한다.

```java
@MappedSuperclass
public abstract class BaseEntity {
    private String createdBy;
    private String lastModifiedBy;
    private LocalDateTime createdDate;
    private LocalDateTime lastModifiedDate;
}
```

> @Entity 클래스는 엔티티나 @MappedSuperClass로 지정한 클래스만 상속이 가능하다.
