# JPA 기본편 - 영속성 전이 CASCADE

#### 영속성 전이 : CASCADE
- 특정 엔티티를 영속 상태로 만들때 연관된 엔티티도 함께 영속성 상태로 만들고 싶을때 사용한다.
- 부모 엔티티를 저장할때 자식 엔티티도 함께 저장

#### 영속성 전이: 저장
```java
@Entity
public class Parent {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(fetch = FetchType.LAZY, mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> children = new ArrayList<>();

    public void addChild (Child child) {
        this.children.add(child);
        child.setParent(this);
    }
}
@Entity
public class Child {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;
}
```

#### 주의
- 영속성 전이는 연관관계 매핑하는 것과 아무 관련이 없다.
- 엔티티 영속화 할때 연관된 엔티티를 함께 영속화 하는 편리함을 제공할 뿐이다.

#### 종류
- ALL
    - 모두 적용
    - 라이프 사이클을 맞출때
- PERSIST
    - 영속
- REMOVE
    - 삭제
- MERGE
- REFRESH
- DETACH

- 게시판 -> 첨부파일
첨부파일 을 여러곳에서 관리할경우 -> 사용해서는 안된다.

다른 엔티티에서도 연관관계를 가지는 엔티티일 경우에는 사용해서는 안된다.

> ALL(라이프사이클을 완전히 일치), PERSIST 정도만 실무에서 쓸만하다. 갑작스레 삭제되거나 하는 일이 생길수 있기때문..

#### 고아 객체
- 고아 객체 제거
    - 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능
- orphanRemoveal = true
```java
Parent parent = em.find(Parent.class, id);
parent.getChild().remove(0);
// 자식 엔티티를 컬렉션에서 제거 했을경우 자동으로 DELETE Query가 나간다.
```
#### 주의
- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 간주하고 삭제하는 기능이다.
- 참조하는 곳이 하나일 때 사용 해야한다.
- 특정 엔티티가 개인 소유할 때 사용 해야한다.
- @OneToOne, @OneToMany 만 가능
- 개념적으로 보마를 제거하면 자식은 고아객체가 된다.
- 따라서 고아 객체 제거 기능을 활성화 시키면, 부모를 제거할때 자식도 함께 제거된다.
- CascadeType.REMOVE 처럼 동작한다.

#### 영속성 전이 + 고아객체, 생명주기
- CascadeType.ALL + orphanRemoval = true
- 스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화, em.remove()로 제거한다.
- 두 옵션을 모두 활성화 하면 부모엔티티를 통해서 자식의 생명주기를 관리할 수 있다.
- 도메인 주도설계 (DDD)의 Aggregate Root 개념을 구현할때 유용하다.
    - repository는 Aggregate Root만 접근이 가능하다 등..
    
> Child의 생명주기는 Parent가 관리한다.
