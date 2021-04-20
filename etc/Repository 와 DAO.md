# Repository 와 DAO

## Repository
- DDD 에서 나온 개념이다.
- **Domain Layer** 에 속한다.
- 객체지향 적인 컬렉션 관리 인터페이스를 제공하기 위해 사용된다.
- Aggregate 당 하나씩 존재한다.
- Order 와 OrderLineItem 이라는 엔티티가 존재할때 이 둘의 라이프사이클이 동일하다면 하나의 Aggregate 로 묶을 수 있다.
- 이렇게 형성된 Aggregate 으로 인해 OrderRepository 하나만으로 OrderLineItem 까지 모두 관리한다.
- 유의할 점은 Repository 도 결국 퍼시스턴스 매커니즘을 사용해야한다. (때문에 의존적이다)
- 이를 해결하기 위해 Repository를 인터페이스와 구현부로 분리한 뒤 인터페이스를 도메인 레이어에 위치시키고, 구현부는 퍼시스턴스 레이어에 위치시킨다.
- 이러한 패턴을 **Seperated Interface** 패턴 이라고한다.

## DAO
- 퍼시스턴스 로직인 Entity Bean 을 대체하기 위해 만들어진 개념
- DAO 는 Persistence Layer 에 속한다.
- Repository 와 마찬가지로 객체지향적인 인터페이스를 제공하려는 의도가 있지만 차이가 있다.
- DAO 는 실질적인 **CRUD 쿼리와 1:1 매핑** 된다는 점이다.
- 일반적으로 데이터베이스 테이블 하나당 하나씩 생성되며, 퍼시스턴스 레이어에 대한 FACADE 역할을 수행한다.

## 정리
- 이 둘의 가장 큰 차이는.. Repository 는 Order 객체와 OrderLineItem 이 메모리에 로드되어 있다고 가정하고, 특정 객체 / 객체 집합에 접근하기 위해 Repository 를 사용한다는 점이다.
- 또한 DAO 는 **TransactionScript Pattern** 과 함께 사용되고, Repository 는 **Domain Model Pattern** 과 함께 사용된다.
- Repository 는 도메인 모델링 과정에서 Repository 가 식별되지만, DAO 는 테이블당 하나씩 생성된다.


## 참고
- http://egloos.zum.com/aeternum/v/1160846


