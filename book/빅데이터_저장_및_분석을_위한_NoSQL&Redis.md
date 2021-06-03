# 빅데이터 저장 및 분석을 위한 NoSQL & Redis

## 1. NoSQL & Redis 소개

### 1.1 NoSQL
- 기존에는 파일 시스템 혹은 DBMS 와 같은 저장 및 관리 기술을 주로 사용해왔지만 이들 만으로는 최근의 데이터양을 처리하는데 한계가 있다.
- 기존 RDBMS 는 클라이언트/서버 플랫폼 기반, NoSQL 은 클라우드, 클라이언트/서버 플랫폼 모두를 기반으로 한다.
- NoSQL 은 SQL 이 아니다 라는 말보단, 기존에 SQL 이 제공하는 기능 뿐 아니라 SQL 이 할 수 없는 영역의 일 까지 가능한 기술 이다.

`장점`
- 클라우드 환경에 적합하다.
    - 오픈 소스이며, Dual Licence 를 제공함
- 유연한 데이터 모델이다.
    - RDBMS 의 경우 분석, 설계, 구측을 위해 3가지 모델링 단계 (개논물) 과 정규화 및 반정규화를 수행 한뒤 테이블을 생성한다.
    - 하지만 NoSQL 은 비정형 데이터 구조 (RDBMS 에 비해 유연함)
- 빅데이터 처리에 효과적이다.
    - NoSQL 은 처음부터 빅데이터를 고려하고 만들어진 SW
    - RDBMS 에 비해 보장된 성능을 보여준다.

### 1.2 NoSQL 종류

`NoSQL 제품`
- MongoDB, Redis, Cassandra, Neo4j 등이 존재한다.
- 이들이 모두 같은 데이터 구조를 사용하지는 않는다.
- Key-Value, Column-Family, Document, Graph 와 같이 4가지 유형의 구조를 제공한다.

> 4가지 유형으로 나뉘는 이유는 데이터의 **유용성 (Availability)**, **일관성 (Consistency)**, **지속성 (Partitioning)** 을 고려한 결과

`빅데이터 데이터 모델링을 위한 가이드 라인`
- 초당 5만건 이상의 데이터가 발생하는가 ?
- 트랜잭션 제어가 필요한가 ?
- 데이터 무결성이 요구되는가 ?
- 수평적 DataSets 인가 ?
    - RDBMS 의 테이블 구조를 2차원 수평적 데이터 구조라고 한다.
- 계층형 데이터인가 ?
    - 일반적인 경우 Document DB 로도 충분히 커버가 가능하지만 Depth 가 10~20을 넘어간다면 Graph DB 를 권장한다.

### 1-3. Key-Value DB 의 활용

`장점`
- In-Memory 기반 데이터 저장구조
- 하나의 Key 와 데이터 값으로 구성됨
- 가공처리가 요구되는 비즈니스 환경에 적합

> Redis 는 대부분 보조 DB 로 많이 사용된다.

`활용 영역`
- 실시간 분석
- IOT 영역
- 계측 정보수집 영역
- 개인화 정보관리 영역
- 전자 상거래 비즈니스 영역

### 1-4. NoSQL 선정 방법
- SimpleAPI 를 지원하는가 ?
- Easy Distributed 기술을 제공하는가 ?
- Easy Replication 이 지원되는가 ?
- Scale-Out 이 가능한가 ?
- 유지보수 비용이 저렴한가 ?
- 비정형 데이터 구조를 지원하고, 트랜잭션 제어가 가능한가 ?
- 오픈소스인가 ?

## 2. Redis 설치 및 데이터 처리

### 2-1. 주요 특징
- Redis 는 Key-Value 데이터베이스로 분류되는 NoSQL이다.
- Key-Value DB 이면서 대표적인 In Memory 기반 데이터 처리 및 저장기술을 제공한다. 이 때문에 빠른 Read/Write 가 가능하다.
- String, Set, Sorted Set, Hash, List, HyperLogLogs 유형의 데이터 저장이 가능
- Dump 파일과 AOF (Append Of File) 방식으로 메모리 상의 데이터를 **디스크에 저장** 이 가능하다.
- Master/Slave Replication 기능을 통해 데이터 분산, 복제 기능을 제공한다.
- **Query Off Loading** 기능을 통해 Master 는 Read/Write, Slave 는 Read 만 수행할 수 있다.
- 파티셔닝을 통해 동적인 스케일 아웃이 가능하다.
- Expiration 기능은 일정 시간이 지난뒤 메모리상 데이터를 자동 삭제할 수 있다.

`Redis 의 주요 업무 영역`
- InMemory DB 의 최대 장점인 빠른 읽고 쓰기가 가능하지만 메인 DB 로 사용하기엔 제한적이다.
- 주된 사용처는 데이터 캐싱, IOT Device 를 통한 데이터 수집 및 처리, 실시간 분석 및 통계
- Message-Queue, 머신러닝, Application Job Management, 검색 엔진 에서 RDB 와 다른 NoSQL 에 비해 효율적인 사용이 가능하다.

### 2-2. 제품 유형
- 다른 NoSQL 과 마찬가지로 듀얼 라이센스를 제공하고 있다.
- 듀얼 라이센스는, 하나의 제품에 2가지 유형의 라이센스가 제공된다는 것이다.
- 이를 용도에 따라 선택할 수 있다.

1. 커뮤니티 에디션
    - 오픈소스 라이선스를 기반으로 개발 및 지원되는 기술
2. 엔터프라이즈 에디션
    - 해당 SW 를 사용하면서 발생하는 기술 문제에 대해 기술 지원을 받을 수 있다.

`Redis 시스템의 전체 아키텍쳐 구성`
- 데이터 저장 엔진
- 분산 시스템
- 복제 시스템
- Index Support
- 관리 툴 (Redis-Server, Redis-Cli, Redis-BenchMark 등)
- 서드파티 모듈을 이용해 더 다양한 활용이 가능함
    - Redis Search Engine : 검색 엔진
    - RedisSQL : SQLite DB 연동
    - RedisGraph : GraphDB 연동
    - Redis sPiped : 암호화

### 2.3 다운로더 및 설치

### 2.4 Redis 시작과 종료

### 2.5 데이터 처리

`용어 설명`
- Table: 하나의 DB 에서 데이터를 저장하는 논리적인 구조
- Data Sets: 테이블을 구성하는 논리적인 단위, 하나의 데이터셋은 **하나의 KEY 와 하나 이상의 FIELD/ELEMENT 로 구성** 된다.
- Key: 하나의 Key 는 하나 이상의 조합된 값으로 표현 가능하다.
- Values: 해당 Key 에 대한 구체적인 데이터 값을 표현한다.

`데이터 입력/수정/삭제/조회`

| 종류 | 설명 |
| --- | --- |
| set | 데이터 저장 |
| get | 데이터 조회 |
| rename | 저장된 데이터 값 변경 |
| randomkey | 저장된 Key 중 하나의 Key 를 랜덤하게 검색 |
| keys | 저장된 모든 Key 를 검색 |
| exists | 검색 대상 Key 존재 여부 확인 |
| mset / mget | 다수의 Key, Value 를 한번에 저장 및 검색 |

`데이터 타입`

| 종류 | 설명 |
| --- | --- |
| strings | 문자, Binary 유형 데이터를 저장 |
| List | 하나의 Key 에 여러 개의 배열 값을 저장 |
| Hash | 하나의 Key 에 여러 개의 Fields 와 Value 로 구성된 테이블을 저장 |
| Set, Sorted Set | 정렬되지 않은 String, Set 과 Hash 를 결합한 타입 |
| Bitmaps | 0 과 1 로 표현하는 데이터 타입 |
| HyperLogLogs | Element 중 Unique 한 개수의 Element 만 계산 |
| Geo | 좌표 데이터를 저장 및 관리 |

> Redis 에서 데이터를 표현하는 기본 타입은 Key 와 Field/Element 를 저장하는 방식 \n
> Key 에는 아스키 값을 저장할 수 있고, Value 는 기본적으로 Strings 와 Container 타입 (Hash, List, Set, Sorted Set) 을 저장할 수 있다.

- **Hash**
    - RDB 에서 PK 와 컬럼으로 구성된 테이블과 매우 유사한 데이터 유형
    - 하나의 Key 는 오브젝트 명과 하나 이상의 필드 값을 콜론 (:) 으로 조합해 표현할 수 있다.
    - 필드 개수의 제한은 없다.
    - hmset, hget, hgetall, hkey, hlen 명령어를 사용한다.
- **List**
    - 일반적인 프로그래밍 언어에서 배열과 유사한 데이터 구조
    - 기본적으로 Strings 타입의 경우 배열에 저장가능한 크기는 **512MB**
    - lpush, prange, rpush, rpop, llen, lindex 명령어를 사용한다.
- **Set**
    - Set 은 여러개의 엘리먼트로 데이터 값을 표현한다. (Object 처럼 표현한다고 생각하면 쉬움)
    - sadd, smembers, scard, sdiff, sunion 명령어를 사용한다.
- **Sorted Set**
    - Set 과 동일한 데이터 구조이고, 데이터 값이 정렬된 상태
    - zadd, zrange, zcard, zcount, zrank, zrevrank 명령어를 사용한다.
- **Bit**
    - Redis 에서 제공되는 Bit 타입은 사용자의 데이터를 0과 1로 표현하여 가장 빠르게 저장 및 해서이 가능하도록 표현하는 구조
    - setbit, getbit, bitcount 명령어를 사용한다.
- **Geo**
    - 위치 정보 데이터를 효율적으로 저장 관리할 수 있는 데이터 구조
    - geoadd, geopos, geodist, georadius, geohash 명령어를 사용한다.
- **HyperLogLogs**
    - RDB 테이블에서 Check 제약조건과 유사한 개념
    - 특정 필드 혹은 엘리먼트에 저장되어야 할 값을 미리 생성하여 저장 한 뒤 필요에 따라 연결하여 사용할 수 있는 데이터 구조
    - pfadd, pfcount, pfmerge 명령어를 사용한다.

### 2.6 Redis 확장 Module

`REJSON`
- Redis 서버에서 JSON 데이터 타입을 저장할 수 있게 해주는 확장 모듈

`REDISQL`
- Redis 서버와 RDB 인 SQLite 와 연동해서 사용할 수 있게 해주는 확장 모듈

`RediSearch`
- Redis DB 에 저장된 데이터에 대해 검색엔진을 사용할 수 있게 해주는 확장 모듈

`Redis-ML`
- 머신러닝 모델 서버를 Redis 서버에서 사용할 수 있게 해주는 확장 모듈

`Redis-sPiped`
- Redis Server 로 전송되는 데이터를 암호화 할 수 있게 해주는 확장 모듈

### 2.7 Lua Function & Script
- Redis Server 에 내장된 Lua Interpreter 를 통해 미리 작성된 Lua Script/Function 을 사용할 수 있다.
- 기본적으로 다양한 Lua Function 을 제공한다.

## 3. 트랜잭션 제어 & 사용자 관리

### 3.1 Isolation & Lock
- Hadoop 과 같은 파일 시스템 기반은 기본적인 단위의 트랜잭션을 제어할 수 없다.
- 이 때문에 제공되는 기술이 NoSQL (하지만 모든 NoSQL 제품이 트랜잭션을 지원하는 것은 아님)
- RDB 의 Commit, Rollback 처럼 트랜잭션 제어가 가능한것 중 하나가 Redis (이를 Read UnCommitted 라고 표현한다.)
- Commit, Rollback 을 지원하면 초당 10만 ~ 20만 이상의 빠른 쓰기/읽기 성능을 보장하기 힘들다.
- 이를 보완하기 위해 Redis 는 Read Committed 트랜잭션 제어도 제공한다.

`DBMS 락 매커니즘`
- Global Lock
- Database Lock
- Object Lock
- Page Lock
- Key/Value Lock (Data Sets Lock)

> Redis 4.0 버전은 데이터-셋 레벨의 락 매커니즘을 제공한다.

### 3.2 CAS (Check And Set)
- 트랜잭션 충돌시 이를 인지할 수 있어야 하는데, 이를 Redis 에서는 CAS (Check And Set) 라고 표현한다.

> Redis 의 Watch 명령어를 통해 다중 트랜잭션이 발생하는지 여부를 모니터링 한다.

### 3.3 commit & rollback
- Redis 는 변경한 데이터를 최종 저장할때 EXEC, 취소시 DISCARD 명령어를 사용한다.

### 3.4 Index 유형과 생성
- Redis 는 기본적으로 하나의 Key 와 Field/Element 값으로 구성된다.
- Key 는 빠른 검색을 위해 기본적으로 인덱스가 생성되는데 이를 **Primary Key Index** 라고 한다.
- 사용자 필요에 따라 추가적으로 인덱스 생성이 가능한데 이를 **Secondary Index** 라고 한다.
- 인덱스 키를 통해 검색시 유일한 값을 검색하는 경우 **Exact Match by a Secondary Index**
- 일정 범위의 값을 검색조건으로 사용하는 경우 **Range by a Secondary Index** 라고 한다.

### 3.5 사용자 생성 및 인증/보안/Roles

`Access Control Privilege`
- 가장 기본적인 접속 권한 중 하나인 액세스 컨트롤
- 미리 사용자 계정 및 암호를 생성해두고, Redis 접근시 인증을 받는 방식
- Standalone Redis 서버 접속시 기본적인 액세스 컨트롤 권한을 부여할 수 있지만, 여러대의 서버로 분산 및 복제 시스템으로 구축하는 경우에는
- Master, Slave, Sentinel, Partition, Replication 서버 간에 **네트워크 액세스 컨트롤 권한** 이 추가로 필요하다.
- 이를 활성화 시키려면 conf 파일내에 requirepass, masterauth 파라메터로 설정해 주어야함

`Authorization Method`
- Redis 는 2가지 인증 방법을 제공한다.
- 1) OS 인증 방법, conf 파일에 접속할 IP 주소를 미리 지정해 둔다.
- 2) Internal 인증 방법, Redis 에 접속한 뒤 auth 명령어로 미리 생성한 계정과 암호로 권한을 부여 받는다.
    
> 커뮤니티 에디션은 Standalone 서버를 구축하는 경우 기본적으로 사용자 계정을 사용자가 직접 생성할 수 없다. \n
> 엔터프라이즈 에디션은 직접 설계 및 생성할 수 있으며, 사용자 정의 롤을 생성하여 부여할 수 있다.



