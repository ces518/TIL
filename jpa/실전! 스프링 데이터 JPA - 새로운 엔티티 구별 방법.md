# 실전! 스프링 데이터 JPA - 새로운 엔티티 구별 방법

#### 새로운 엔티티 구별 방법
`기본 전략`
- 식별자가 객체일때 `null`로 구분
- 식별자가 자바 기본 타입일때 `0` 으로 판단
- `Persistable` 인터페이스를 구현해서 엔티티 구별 로직 변경 가능

`SimpleJpaRepository`
```java
@Transactional
@Override
public <S extends T> S save(S entity) {

    if (entityInformation.isNew(entity)) { // 새로운 객체 일경우 
        em.persist(entity); // 엔티티에 Id 식별자를 생성해줌
        return entity;
    } else {
        return em.merge(entity);
    }
}
```

#### 정리
- JPA 식별자 생성전략이 @GeneratedValue 를 사용하면 save() 호출 시점에 식별자가 없으므로 새로운 엔티티로 이식하고, 정상 동작한다.
- 하지만 @GeneratedValue를 사용하지 않고, 채번 테이블등을 사용해서 직접 할당하는 경우 기본 전략에 의해 기존 엔티티로 인식한다.
- DB에 있는지 먼저 조회를 한뒤, INSERT 쿼리가 발생한다. (비 효율적)

> 만약 채번 테이블등을 사용해서 식별자를 직접 할당하는 경우 Persistable<ID> 인터페이스를 구현 해주어야 한다.

```java
@EntityListeners(AuditingEntityListener.class)
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {

    @Id @GeneratedValue
    private String id;

    @CreatedDate
    private LocalDateTime createdAt;

    public Item(LocalDateTime createdAt) {
        this.createdAt = createdAt;
    }

    @Override
    public boolean isNew() {
        // 생성일이 null 일 경우 새로운 엔티티로 판단하도록 엔티티 식별 로직을 구현
        return createdAt == null;
    }
}
```
