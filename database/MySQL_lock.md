# MySQL InnoDB 락의 종류

### Shared Lock (S)
- Row-Level Lock
- SELECT를 위한 read lock
- shared lock 이 걸려있는 동안 다른 트랜잭션에서 해당 row 에 대해 shared lock 획득이 가능하다.

### Exclusive lock (X)
- Row-Level Lock
- UPDATE, DELETE 를 위한 write lock
- exclusive lock 이 걸려있다면 다른 트랜잭션이 해당 row에 대해 X, S lock을 모두 획득하지 못하고 대기해야 한다.

### Intention lock
- Table-Level Lock
- 테이블 내 Row에 대해서만 나중에 어떤 row-level 락을 걸 것이라는 의도를 알려주기 위해 미리 table-level에 걸어두는 lock
- SELECT .. 가 실행되면 Intention shared lock 이 테이블에 걸리고, 그 후 row-level shared lock이 걸린다.
- SELECT FOR UPDATE, INSERT, DELETE, UPDATE 가 실행되면 Intention exclusive lock 이 테이블에 걸리고, 그 후 row-level에 exclusive lock 이 걸린다.
- LOCK TABLES, ALTER TABLE, DROP TABLE 이 실행될 때 위의 두 Lock 모두 block하는 table-level lock 이 걸린다.

### 2단계에 거쳐서 Lock 을 거는 이유 ?
- A 트랜잭션에서 table-level lock 이 걸려있는데 B 트랜잭션에서 row-level lock 을 거는것을 방지
- row-level write 가 일어나는 도중 테이블 스키마가 변경되는 일 방지 

# 락이 적용되는 상황에 따른 분류

### Record Lock
- PK, UNIQUE-INDEX 로 조회하여 하나의 인덱스 레코드에만 lock을 거는 것
- row-level lock 이란 인덱스 레코드에 대해 락을 거는 것

### Gap Lock = Range Lock
- 실제 존재하는 인덱스 레코드에 락을 거는 것이 아닌, 범위를 지정하기 위해 인덱스 레코드 사이의 범위에 락을 건다.
- SELECT 쿼리를 두번 실행햇을때 다른 트랜잭션에서 데이터가 수정되더라도 같은 결과가 리턴되는것을 보장한다. (팬텀 리드 방지)
- 인덱스가 존재하지 않는다면, Gap Lock을 수행한다.

### Next-Key Lock
- 범위 지정한 쿼리를 실행하면, record lock 과 gap lock을 복합적ㅇ로 수행한다.


#### 참고
- https://www.letmecompile.com/mysql-innodb-lock-deadlock/