# HashMap의 element 저장방식

- Bucket, Load factor 라는 개념이 존재한다.
- capacity = bucket크기
- HashMap 에 저장된 데이터수 == capacity*load_factor
- capacity의 기본값은 16
- load_facotr의 기본값은 0.75


### [index] 공식

```java
int index = key.hashCode() % capacity;
```

### 버킷의 크기가작다면 ?

- index가 중복되는위치에 seperate channing이 발생하여 linked-list 형태로 같은위치에 저장하게된다.
- 검색시 해당위치에 linked-list형식으로 여러개의 값이 존재할경우 해시코드를 이용한 equals 연산에 의해 속도가 저하됨.
- 이를 방지하고자 HashMap은 자신의 capacity에 일정수 이상 데이터 차지시 자동으로
- bucket크기를 늘림 [약 2배]
  - 늘리는 시점 ?
  - load_factor == 저장된 데이터 수 / capacity  가 되는시점에 늘림.


### 그렇다면 버킷의 크기를 무작정 크게 잡는것이 좋을까 ?
- 처음부터 capacity를 너무 크게잡는것도 능사는아님.
- 빈 공간이 많을경우 메모리낭비 + interation(객체모든데이터조회) 상황시 성능저하.

### load factor를 기준으로 살펴보자
- load factor가 작으면, 그만큼 capacity가 커지므로 메모리는 많이 차지하지만, 검색 속도가 빨라집니다.
- load factor가 커지면 그만큼 capacity가 덜 커지므로 메모리는 적게 차지하지만 검색 속도는 느려집니다.
- API에서는 가장 이상적인 값을 0.75라고 말하고 있습니다.
- 이 값은 메모리와 실행성능을 모두 고려했을 때 최적의 값입니다.
- 하지만 만약 자신이 개발하는 애플리케이션이 ‘메모리는 좀 많이 잡아먹어도 괜찮으니 검색속도가 빨랐으면 한다’
- 라고 하면 load factor를 작게 설정할것.


### 언제 capacity를 설정해주어야 할까 ?
- 데이터 수 짐작 가능시 capacity 설정 해주는것이 좋음.
- 만약 16만개데이터가 들어오는데 capacity가 16이라면 증가작업이 상당수일어날것.


### capacity의 증가가 되었을때 어떤일이 발생할까 ?
- capacity 증가가 bucket의 크기만 증가 시키고 끝나는게 아닙니다.
- capacity 값이 변경됬으니, 기존에 저장되어 있던 모든 데이터의
- hashCode() % capacity 값을 다시 계산하고,
- 그 값에 따른 공간 배치를 새로 다 해야합니다. 이 과정을 rehashing이라고 하는데 이 과정이 부하가 엄청나게 일어난다.
- 따라서 자료의 수가 많다고 판단되는경우 capacity 의 크기를 지정해주는것이좋다.




