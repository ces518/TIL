# 지옥 스터디 - 14 모르는 것에 대한 두려움

## 목표 : 누락된 값을 구분하기
- 데이터베이스 어떤 데이터에 **값이 없는것** 은 피할 수 없다.
- 모든 칼럼에 대한 값을 알기 전에 행을 삽입해야 할 수도 있고, 또는 일부 칼럼은 어떤 상황에서는 의미 있는 값을 가지지 않을 수도 있다.
- SQL 은 특수한 값인 NULL 을 지원한다.
- SQL 테이블과 쿼리에서 NULL 을 생산적으로 사용할 수 있는 다양한 방법이 있다.
  - 여전히 일하고 있는 직원의 퇴사일
  - 전기만 사용하는 자동차에 대한 연료 효율같이 칼럼에 주어진 행에서만 적용 가능한 경우
  - 함수에 인수로 DAY ('2009-12-32') 같이 유효하지 않은 값이 입력된 경우
  - 외부 조인에서 매치되지 않은 행의 칼럼 값의 자리를 채우는 용도

> 목표는 NULL 을 포함하는 컬럼에 대한 쿼리를 작성하는 것

## 안티패턴 : NULL 을 일반 값 처럼 사용
- 많은 개발자가 SQL 의 NULL 동작을 보고 당황한다.
- SQL 에서는 NULL 이 0이나 false, 혹은 빈 문자열과 다른 특별한 값으로 취급한다.

### 수식에서 NULL 사용
- NULL 값을 가지는 칼럼이나 수식에 대해 산술 연산을 수행했을 경우 이다.

```sql
SELECT hours + 10 FROM Bugs;
```
- NULL 에 10 을 더하는 경우 NULL 이 반환된다.
- NULL 은 0과 같지 않다.

### NULL 을 가질 수 있는 칼럼 검색
- 다음 쿼리는 assigned_to 값이 123인 행만을 리턴하고, 다른 값을 가지거나 칼럼이 NULL 인 행은 리턴하지 않는다.

```sql
SELECT * FROM Bugs WHERE assigned_to = 123;
```
- 다음 쿼리는 위 쿼리의 여집합이 리턴될꺼라 생각 한다.

```sql
SELECT * FROM Bugs WHERE NOT (assigned_to = 123);
```
- 하지만 두 쿼리 모두 칼럼 값이 NULL 인 행은 리턴하지 않는다.
- NULL 인 행을 찾을때 다음과 같은 실수를 많이 하낟.

```sql
SELECT * FROM Bugs WHERE assigned_to = NULL;
SELECT * FROM Bugs WHERE assigned_to <> NULL;
```
- 위 두 쿼리는 NULL 인 행을 리턴하지 않는다.

### 쿼리 파라미터로 NULL 사용
- 파라미터를 받는 SQL 에서 NULL 을 다른 값처럼 사용하기가 어렵다.

```sql
SELECT * FROM Bugs WHERE assgiend_to = ?;
```
- 위 쿼리는 NULL 을 파라미터로 사용할 수 없다.

### 문제 회피하기
- NULL 을 조작하는 것이 쿼리를 더 복잡하게 만드는 경우 많은 개발자가 NULL 을 허용하지 않도록 한다.
- 예를 들어 NULL 이 아닌 -1 과 같은 값을 사용하는 것이다.
- 하지만 -1 은 SUM(), AVG() 와 같은 계산시 포함된다.
- 별도 로 이런 값을 가진 행을 예외 처리를 해야 한다.

## 안티패턴 인식 방법
- 애플리케이션에서 몇몇 사용자의 이름이 표시되지 않아.
- 이 프로젝트의 전체 작업시간 보고서에 우리가 완료한 몇몇 버그만 포함되어 있어. 우선순위 할당한 것들만 포함되어 있네.
- Bugs 테이블에서 알수없음을 나타내던 문자열을 이제 사용할 수 없다.

## 안티패턴 사용이 합당한 경우
- NULL 을 사용하는 것은 안티패턴이 아니다.
- NULL 을 일반적인 값처럼 사용하는 것이 안티패턴이다.

## 해법 : 유일한 값으로 NULL 을 사용하라
- NULL 값과 관련된 대부분의 문제는 SQL 의 세 가지 값 로직의 동작을 제대로 이해하지 못한데서 비롯된다.
- 대부분 다른 언어에 익숙하다면 약간 어렵게 느껴질 수도 있지만 조금만 공부하면 쉽게 NULL 을 잘 다룰 수 있다.

### 스칼라 수식에서의 NULL

| 수식 | 기대 값 | 실제 값 | 이유 |
| --- | --- | --- | --- |
| NULL = 0 | TRUE | NULL | NULL 은 0이 아니다. |
| NULL = 12345 | FALSE | NULL | 지정된 값이 모르는 값과 일치하는지 알 수 없다. |
| NULL <> 12345 | TRUE | NULL | 또한 다른지도 알 수 없다. |
| NULL + 12345 | 12345 | NULL | NULL 은 0이 아니다. |
| NULL + 'string' | 'string' | NULL | NULL 은 빈문자열이 아니다 |
| NULL = NULL | TRUE | NULL | 모르는 값과 모르는 값이 일치하는지 알 수 없다. |
| NULL <> NULL | FALSE | NULL | 또한 다른지도 알 수 없다. |

### 불리언 수식에서의 NULL
- 불리언 수식에서 NULL 의 동작을 이해하기 위한 핵심 개념은 NULL 도 true 도 false 도 아니다.

| 수식 | 기대 값 | 실제 값 | 이유 |
| --- | --- | --- | --- |
| NULL AND TRUE | FALSE | NULL | NULL 은 false 가 아니다. |
| NULL AND FALSE | FALSE | FALSE | FALSE 와 AND 는 FALSE. |
| NULL OR FALSE | FALSE | NULL | NULL 은 false 가 아니다. |
| NULL OR TRUE | TRUE | TRUE | TURE 와 OR 는 TRUE |
| NOT (NULL) | TRUE | NULL | NULL 은 false 가 아니다.|

### NULL 검색하기

```sql
SELECT * FROM Bugs WHERE assigned_to IS NULL;
SELECT * FROM Bugs WHERE assigned_to IS NOT NULL;
```
- SQL-99 표준에서는 IS DISTINCT FROM 이라는 비교 연산자가 정의되었다.
- 피연산자가 NULL 이 더라도 항상 true/false 를 리턴한다.

```sql
SELECT * FROM Bugs WHERE assgined_to IS NULL OR asggiend_to <> 1;
SELECT * FROM Bugs WHERE assgiend_to IS DISTINCT FROM 1;
```
- IS NULL 로 확인해야 하는 수식을 사용하지 않아도 된다.
- 쿼리 파라미터로 NULL 을 사용하는 것이 가능해 진다.
- MySQL 은 이처럼 동작하는 전용 연산자 <=> 를 제공한다.

### 칼럼을 NOT NULL 로 선언하기
- NULL 값이 애플리케이션 정책을 위반하거나 또는 의미가 없는 경우에는 칼럼에 NOT NULL 제약조건을 사용하는게 권장 사항이다.
- 애플리케이션 코드에 의존하기 보다 데이터베이스가 제약조건을 균일하게 강제하는 것이 좋다.

### 동적 디폴트
- 어떤 쿼리에서는 쿼리 로직을 단순화하기 위해 칼럼이나 수식이 NULL 이 되지 않게 강제할 필요가 있다.
- 그 값이 저장되지 않기를 발라 수 있는데. 주어잔 칼럼이나 수식에 특정 쿼리에서만 디폴트 값을 설정하는 방법이다.
- 이를 위해 COALESCE() 를 사용할 수 있다.

```sql
SEELCT first_name || COALESCE(' ' || middle_initial || ' ', ' ') || last_name AS full_name
FROM Accounts;
```

> 어떤 데이터 타입에 대해서든 누락된 값을 뜻하는데 NULL 을 사용하라.
