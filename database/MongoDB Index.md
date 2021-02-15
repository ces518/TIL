# MongoDB Index

## Index 란 ?
- 책의 인덱스와 데이터베이스의 인덱스의 공통점 ?
  - 핵심된 키워드
  - 정렬되어 있음
- 데이터베이스에서 테이블에 대한 동작 속도를 높혀주는 자료구조
- 특정 필드를 기준으로 정렬해서 보관한다.
- Write 성능을 손해보는 대신, Read 성능을 향상

## MongoDB Index 의 종류
- **Regular Index 일반적인 인덱스**
- Geospatial Index 지구 공간정보
- Text Index 문자열에 대한 텍스트 검색 쿼리
- Hashed Index 해싱된 값 저장
- Mutikey Index 배열의 각 요소에 대한 검색
- TTL Index 보관기간이 지난 도큐먼트 자동 삭제
> MongoDB 의 Index 는 모두 B+ Tree 로 구현되어 있다.

## B - Tree
- 자식 노드가 2개 이상인 트리
- 이진 트리의 확장 개념이다.
- 하나의 노드에 다수의 값을 저장하여 검색 성능을 향상시킨다.
- 순차 검색 효율이 낮다.

## B + Tree
- B - Tree 의 단점을 보완
- Leaf Node 가 Linked List 로 연결된 구조
- 모든 데이터는 Leaf 노드에 저장된다.
  - Root 와 Branch Node 는 Key 의 역할만 수행한다.
- 랜덤 검색, 순차 검색의 절충안이다.
- 균등한 응답속도를 보장한다.
- 노드의 접근 횟수를 감소시킨다.
> DBMS 에서 가장 널리 사용되는 인덱스 자료구조
