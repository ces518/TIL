# 실전 스프링부트와 JPA 활용 1 - 웹애플리케이션 개발 - 상품 서비스 개발

#### 상품 서비스 개발
```java
/**
 * 상품 서비스는 사실상 repository로 위임만 하는 역할을 한다.
 * 이런 경우에는 서비스를 따로 만들지 않고, Controller 에서 바로 호출하는것을 고려해보는것이 좋다.
 */
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class ItemService {

    private final ItemRepository itemRepository;

    // 메소드에 가까이 위치하는 @Transactional 설정이 우선권을 가진다.
    @Transactional
    public void saveItem (Item item) {
        itemRepository.save(item);
    }

    public Item findOne (Long itemId) {
        return itemRepository.findOne(itemId);
    }

    public List<Item> findItems () {
        return itemRepository.findAll();
    }
}
```

- @Transactional 애노테이션은 Interface > Class > Method 순으로 가까울수록 설정 우선권을 가진다.

> 상품 서비스는 repository로 위임만 하는 역할을 하는데, 이런 구조의 경우 서비스를 따로 만들지않고, 직접 호출하는 것을 고려해보는것이 좋다.
