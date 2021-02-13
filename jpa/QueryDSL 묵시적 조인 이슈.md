# QueryDSL 묵시적 조인 이슈
- QueryDSL 에서 묵시적 조인을 사용하면 발생하는 이슈
- 묵시적 조인을 사용하면, 일반적으로 내부 조인을 사용한다. 라고 JPA 에 설명되어 있지만 QueryDSL 의 경우에는 조금 다르다.
- QueryDSL 에서 크로스조인과 내부조인을 둘다 사용할 수 있는 상황이라면, 크로스조인을 우선으로 사용한다.
- 이는 QueryDSL 에 자주올라오는 이슈중 하나이다.

## 참조
- https://github.com/querydsl/querydsl/issues/1833