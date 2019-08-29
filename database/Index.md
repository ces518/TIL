# Index
- DB 테이블의 컬럼값과 해당 레코드가 저장된 주소를 key-value 쌍으로 인덱스를 만들어 주어진 순서로 미리 정렬하여 파일을 생성해 두는것.

- 특성
    - 읽기는 빠르게, 쓰기는 느리게
    - 인덱스 내부 요소는 항상 정렬 되어있어야 한다.
    - SELECT 는 빠르지만, INSERT, UPDATE, DELETE 는 느리다.

#### Index 역할별 종류
- Primary Key
    - 식별자
    - 해당 레코드를 대표하는 컬럼값으로 만들어진 인덱스
    - NULL, 중복 허용하지 않음

- Secondary Key
    - Primary Key를 제외한 나멎 ㅣ인덱스
    - Unique Key는 Primary Key와 성격이 비슷해 대체키라고도 하지만 보조키로 분류하기도 한다.

* 데이터 중복 여부에 따라 Unique Index, Non-unique Index로 구분한다.
* 단순하게 생각하면 유일성 여부를 의미하지만, DBMS 쿼리 실행시 옵티마이저에게는 상당히 중요한 정보.
