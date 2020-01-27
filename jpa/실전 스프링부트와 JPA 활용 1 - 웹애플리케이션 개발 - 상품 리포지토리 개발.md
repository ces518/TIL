# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 상품 리포지토리 개발

#### 상품 리포지토리 개발
```java
@Repository
@RequiredArgsConstructor
public class ItemRepository {

    private final EntityManager em;

    /**
     * 아이템 저장
     * @param item
     */
    public void save (Item item) {
        if (item.getId() == null) {
            // 최초 저장시 새로이 등록
            em.persist(item);
        } else {
            // 기존에 존재하는 엔티티를 업데이트(실제 업데이트는 아니지만 비슷한 느낌이다.)
            em.merge(item);
        }
    }

    /**
     * 아이템 단건 조회
     * @param id
     * @return
     */
    public Item findOne (Long id) {
        return em.find(Item.class, id);
    }

    /**
     * 아이템 목록 조회
     * @return
     */
    public List<Item> findAll () {
        return em.createQuery("select i from Item i")
                .getResultList();
    }
}
```

- 아이템을 저장할때 id 값이 null이라면 새로운 상품 등록이기 떄문에 em.persist() 를 호출한다.
- 만약에 id 값이 null 이 아니라면 기존에 존재하던 상품을 수정하는 것이기 때문에 em.merge() 를 호출한다.
    - em.merge() 는 update와 비슷한 느낌으로 이해하면 된다.
    - 물론 update와 동일하다는 것은 아니다.
