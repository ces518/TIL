# SRE 사례 소개

## Redis Cluster Rebalancing
- Redis 0-16,383 범위의 16,384 개의 hash slot 을 가지고 있고
- slot range 를 각 샤드들이 나눠갖는 구조이다.

- Rebalancing 실행
    - 안전성이 중요한 경우 slot 별로 명령을 실행해서 옮길것
- Rebalancing 명령이 중지된 경우
    - cluster fix 명령어로 slot 복구 가능
- ASK Error
    - slot migration 중 발생할 수 있음
    - 현재 노드에 해당 key 가 없다면 마이그레이션 되는 노드에 확인 Noti 예외
    - Lettuce 의 **버그** 로 인해 발생했음
    - 멀티 스레드 환경에서 Lettuce 가 ASKING 명령을 수행하는 과정에서 발생함
    - Redis 노드당 하나의 커넥션을 공유한다. (여러 스레드가 동시에 접근 가능함)    
- Rebalancing 시 주의
    - big keys 를 확인해 검증이 필요하다.
    - 시뮬레이션이 가능하다면 dump 떠서 시뮬레이션 시도
    - lettuce version 5.3.1.RELEASE 이상
        - ASK 관련해서 커맨드가 중복으로 실행되는 문제가 있음

## Memcached Redistribute
- 4대의 멤캐시 운영중 1대에서 장애가 발생한 상황 (Redistribute 옵션 사용중)
- 대부분의 요청은 재분배된 노드로 가서 정상화 되었으나 일부 요청들은 재분배 되지 않아 에러가 발생
- Memcached Client 의 동작
- 재분배를 시도하더라도 0.78% 확률로 같은 dead node 를 가리키게 된다.
    - 간헐적으로 장애가 발생할 수 있음
    - Ketama hash 알고리즘
    
## Prometheus Scraping
- 특정 서버에서 멤캐시 타임아웃이 발생하기 시작했음
- 멤캐시 타임아웃은 500ms / 보통 1ms 안에 응답이 오기 때문에 장애 의심을 해보아야한다.
- 모니터링 해본 결과 GC 패턴의 이상이 감지됨
> 프로메테우스 메트릭 수집으로 인해 멤캐시 GC 에 문제가 발생했었다 !! \n
> 메트릭의 용량이 MB 단위 였음.. 10MB 인 경우도 존재했다.
- 불필요한 메트릭 제거후, KB 단위로 감소 되었고, GC 패턴이 정상으로 돌아옴
- 문제의 원인은 Humongous Allocation
- 큰 오브젝트로 인해 GC 패턴을 바꾸어버림
- Humongous Object 가 힙사이즈의 1/2 이상 차지하면 바로 Old 영역으로 할당됨
- Humongous Object 를 지속적으로 할당하다보니 Old 영역 부족으로 인해 Full GC 가 발생하기도 함

`해결 방안`
- 메트릭 개수/ 사이즈 제한
- 힙 리전 사이즈 늘리기
- 메트릭을 쪼개서 스트림 방식으로 Response

`문제 분석을 위한 것들`
- Change Log
    - 배포/ 인프라/ 네트워크 등등
- Log Message
    - 문제의 원인을 해결하기 위한 실마리
- 메트릭
    - 많이 남길수록 좋다.
