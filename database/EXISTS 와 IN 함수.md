# EXISTS 와 IN 함수

```sql
-- IN 을 활용한 SQL, 일반적으로 많이 쓰이는 쿼리이다.
-- 데이터가 단건일때는 크게 문제가 되지 않지만, 데이터수가 점점 많아질수록 성능이 기하급수적으로 느려진다.
-- 후에 설명한 EXISTS와 다르게 IN 은 모든 결과값을 비교하기 때문에 성능차가 존재한다.
SELECT A.COL1, A.COL2, B.COL1, B.COL2
FROM TABLE1 A JOIN TABLE2 B
  ON A.ID = B.ID
WHERE A.COL1 IN (SELECT COL1
                 FROM TABLE3
                 WHERE COL2 = 'B')

-- EXISTS를 활용한 SQL, IN 을 대체하는 쿼리이다.
-- IN과는 다르게 해당 로우의 존재여부만 체크하기때문에 내부적으로 돌아가는 방식이나 성능상 크게 차이가 난다.
-- 데이터가 단건일 경우 크게 차이가 나지 않지만, 데이터수가 점점 많아질수록 성능 차이가 극심해진다.
SELECT A.COL1, A.COL2, B.COL1, B.COL2
FROM TABLE1 A JOIN TABLE2 B
  ON A.ID = B.ID
WHERE EXISTS (SELECT 1
                FROM TABLE3 C
                WHERE C.COL2 = 'B'
                AND A.COL1 = C.COL1)
```

> IN과 EXISTS는 대체하여 쓸 수 있지만 NOT IN과 NOT EXISTS는 NULL 처리 방식이 다르기 때문에 조심해서 써야한다.

#### 정리
- IN(서브쿼리)절보단 EXISTS를 이용할 수 있다면 되도록 EXISTS로 대체할것.
    - 서브쿼리가 없는 단순 IN 절은 IN이 훨씬 빠르다.
- 조건절에는 가공될 수 있는 서브쿼리는 넣으면 좋지 않다. 되도록 고정된 컬럼과 비교할 수 있는 조건절을 사용할것.
- IN, EXISTS 이전에 JOIN으로 해결할 수 있다면 JOIN으로 풀어낼것.
