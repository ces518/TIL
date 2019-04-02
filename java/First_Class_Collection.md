# 일급컬렉션 (First Class Collection)

- 소트웍스 앤솔로지의 객체지향 생활체조 파트에서 언급이된 내용이다.
```
이 규칙의 적용은 간단하다. 
콜렉션을 포함한 클래스는 반드시 다른 멤버 변수가 없어야 한다. 
각 콜렉션은 그 자체로 포장돼 있으므로 이제 콜렉션과 관련된 동작은 근거지가 마련된셈이다.
필터가 이 새 클래스의 일부가 됨을 알 수 있다. 
필터는 또한 스스로 함수 객체가 될 수 있다. 
또한 새 클래스는 두 그룹을 같이 묶는다든가 그룹의 각 원소에 규칙을 적용하는 등의 동작을 처리할 수 있다. 
이는 인스턴스 변수에 대한 규칙의 확실한 확장이지만 그 자체를 위해서도 중요하다. 
콜렉션은 실로 매우 유용한 원시 타입이다. 
많은 동작이 있지만 후임 프로그래머나 유지보수 담당자에 의미적 의도나 단초는 거의 없다. - 소트웍스 앤솔로지 객체지향 생활체조편
```
```

// 해당코드를 아래와같이 Warpping하는것을 의미한다.
Map<String,String> map = new HashMap<>();
map.put("key1",1);
map.put("key2",2);
map.put("key3",3);

public class MyCollection {
    private Map<String,String> map;
    
    public MyCollection(Map<String,String> map) {
        this.map = map;
    }
}
```

Collection을 Wrapping하면서 그외 다른 멤버변수가 없는 상태를 말한다.

- 장점 ?
    1. 비지니스에 종속적인 자료구조
    2. Collection의 불변성 보장
    3. 상태와 행위를 한 곳에서 관리
    4. 이름이있는 콜렉션
    


#### 1.비지니스에 종속적인 자료구조
로또복권 프로그램을 개발

- 조건
    1. 번호는 6개만 존재해야한다.
    2. 6개의 번호는 서로 중복이 되지 않아야한다.

- 일반적인 구현    
```java
public class LottoService {
    private static final int MAX_NUMBER_SIZE = 6;
    
    public void createNumber() {
        List<Integer> numbers = createNonValidateNumbers();
        validateSize(numbers);
        validateNumbers(numbers);
    }
    
    private void validateSize(List<Integer> numbers) {
        if(numbers.size != MAX_NUMBER_SIZE) {
            throw new IllegalArgumentException("로또 번호는 6개만 가능합니다.");
        }
    }
    
    private void validateNumbers(List<Integer> numbers) {
        Set<Integer> checkNumbers = new HashSet<>(numbers);
        if(checkNumbers.size() != MAX_NUMBER_SIZE) {
            throw new IllegalArgumentException("번호들은 중복될 수 없습니다.");
        }
    }
}
```
- 서비스메서드에서 모든 비지니스 로직을 처리할 경우 큰 문제가 발생한다.
- 로또번호가 필요한 모든 장소에서 검증 로직이 필요...
- 중복의 발생..

```java
public class LottoCollection {
    private final List<Integer> numbers;
    
    public LottoCollection(List<Integer> numbers) {
        validateSize(numbers);
        validateNumbers(numbers);
        this.numbers = numbers;
    }
    
    private void validateSize(List<Integer> numbers) {
        if(numbers.size != MAX_NUMBER_SIZE) {
            throw new IllegalArgumentException("로또 번호는 6개만 가능합니다.");
        }
    }
    
    private void validateNumbers(List<Integer> numbers) {
        Set<Integer> checkNumbers = new HashSet<>(numbers);
        if(checkNumbers.size() != MAX_NUMBER_SIZE) {
            throw new IllegalArgumentException("번호들은 중복될 수 없습니다.");
        }
    }
}

```
- 비지니스 로직이 모두 일급객체안에 존재함으로써
- 로또번호가 필요한 모든 장소에서 검증로직의 필요성 제거
- 이런 자료구조를 만드는것이 일급 컬렉션


#### 2.불변자료구조

일급 컬렉션은 컬렉션의 불변을 보장한다.

다음과 같이 Java의 final키워드는 불변을 보장하는것이아닌 재 할당만 금지한다.
```java
@Test
public void finalTest() {
    //given
    final List collection = new ArrayList<>();
    //when
    collection = new ArrayList<>(); // 컴파일에러 발생
    //then
    ...
}
```

객체의 불변성을 보장하려면 일급 컬렉션과 래퍼 클래스등의 방법으로 해결해야한다.
```java
public class Gifts{
    private final List<Long> gifts;
    
    public Gifts(List<String> gifts) {
        this.gifts = gifts;
    }
    
    public long getGiftSum() {
        return gifts.stream().sum();
                   
    }
}
```

- 위 의 클래스는 생성자와 getGiftSum 외엔 메서드가 존재하지않는다.
- List컬렉션에 접근할 방법이 없기때문에 불변성을 보장한다.


#### 3. 상태와 행위를 한곳에서 관리

일급 컬렉션의 장점은 값과 로직이 함께 존재한다는것이다.
이는 Enum의 장점과도 동일하다.

- 여러 지역의 값들이 모여있고 특정 지역의 합만 필요하다는 상황
    - 값을 계산하는 로직이 여러곳에서 중복발생
    - 로직 변경시 모든곳을 다 변경해주어야한다.
    - 서울 이외의 지역의 총합도 필요하다면 코드가 흩어질 가능성이 높다.
```java
// 일반적인 상황
// 값과 로직이 분리되어있다.
@Test
public void outerLogic() {
    // given
    List<LocalStatistic> statistics = Arrays.asList(
            new LocalStatistic("seoul",1000),
            new LocalStatistic("busan",1000),
            new LocalStatistic("seoul",1000),
            new LocalStatistic("daejeon",1000));
    // when
    Long seoulSum = statistics.stream()
                                .filter(local -> local.getLocalType().equals(SEOUL))
                                .mapToLong(LocalStatistic::getCount)
                                .sum();
    ...
}
```

- 일급 컬렉션으로 해결
```java
public class LocalStatisticsGroups {
    private List<LocalStatistic> statistics;
    
    public LocalStatisticsGroups(List<LocalStatistic> statistics) {
        this.statistics = statistics;
    }
    
    public Long getSeoulSum() {
        return statistics.stream()
                .filter(local -> LocalType.isSeoul(local.getLocalType()))
                .mapToLong(LocalStatistic::getCount)
                .sum();
    }
}

// 여러 지역에대한 합이 필요할 경우 리팩토링
public class LocalStatisticsGroups {
    private List<LocalStatistic> statistics;
    
    public LocalStatisticsGroups(List<LocalStatistic> statistics) {
        this.statistics = statistics;
    }
    
    public Long getSeoulSum() {
        return getSumByLocal(local -> LocalType.isSeoul(local.getLocalType()));
    }
    
    private Long getSumByLocal(Predicate<LocalStatistic> predicate) {
        return statistics.stream()
                .filter(predicate)
                .mapToLong(LocalSTatistic::getCount)
                .sum();
    }
}
```

- LocalStatisticsGroups 라는 일급 컬렉션이 생김으로써 상태와 로직이 한곳에서 관리된다.


#### 4. 이름이있는 컬렉션 
- 변수명만으로는 의미를 부여하기 힘든것이 현실.
- 각각의 일급 컬렉션을 만듦으로써 이 컬렉션기반의 용어사용 , 검색을 사용하면된다.
