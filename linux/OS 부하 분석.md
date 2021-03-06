# OS 부하 분석

## 부하란 ?
단일 호스트의 성능을 끌어내기 위해서는 서버 리소스의 현황을 정확하게 파악해야 한다.\
이런 부하 계측 작업은 단일 호스트의 부하를 줄이는 데 가장 중요한 작업이다.

> "추측하지 말라, 계측하라."

부하 계측을 위한 대상을 보통 Apache, MySQL 과 같은 애플리케이션이 되기 쉬운데, 이는 잘못된 것이다.\
대상이 되어야 하는것은 그 하위 계층인 **OS** 이다.\
부하를 알기 위해 필요한 거의 **모든 정보는 OS 리눅스 커널** 이 가지고 있다.\
리눅스에서는 ps, top, sar 등의 툴을 사용해서 이를 계측한다.

`top`

```shell
PID   COMMAND      %CPU TIME     #TH   #WQ  #PORT MEM    PURG   CMPRS  PGRP PPID
465   iTerm2       15.0 00:14.86 13    7    364   83M-   6048K+ 17M-   465  1
133   WindowServer 12.8 44:16.28 14    5    2646- 681M-  30M+   101M   133  1
6454  top          4.4  00:00.57 1/1   0    26    3956K+ 0B     0B     6454 6378
733   dsAccessServ 2.5  01:14.06 10    1    92    123M   0B     113M   69   69
4404  idea         2.1  15:38.13 78    3    628   1529M  5576K  630M   4404 1
...
```
- top 은 특정 순간의 OS 상태의 스냅샷을 표시하는 툴
- CPU 사용률, 메모리 사용 현황 등 다양한 값을 알 수 있다.

## 병목 규명작업의 기본적인 흐름
1. Load Average 확인
2. CPU, I/O 중 병목 원인 조사

### Load Average 확인

top 이나 uptime 과 같은 툴로 Load Average 를 확인한다.\
Load Average 는 시스템 전체의 부하 상황을 나타내는 지표지만 병목의 원인을 판단할 수는 없다.

> Load Average 는 낮지만 시스템 전송량이 오르지 않는 경우도 가끔 존재한다.

### CPU, I/O 중 병목 원인 조사

Load Average 가 높다면 다음으로 CPU, I/O 중 원인을 파악해야 한다.\
sar 이나 vmstat 으로 시간 경과에 따른 CPU 사용률 혹은 I/O 대기율의 추이를 확인한다.\

`CPU 부하가 높은 경우`

- 프로그램 처리가 병목인지, 시스템 프로그램이 원인인지 확인 => top / sar
- 프로세스의 상태, CPU 사용시간 등을 보면서 원인이 되는 프로세스 탐색 => ps
- 프로세스를 찾은 후 상세하게 조사할 경우 strace 또는 oprofile 을 사용

일반적으로 CPU 에 부하가 걸리는 경우는 다음 과 같다.

1. 디스크나 메모리 용량 등 그 밖의 부분에서는 병목이 되지 않는 상태
2. 프로그램이 폭주하여 CPU 에 필요이상의 부하가 걸리는 상태

> 1의 경우 서버 증설이나 프로그램 로직을 개선해야 한다.\
> 2의 경우 오류를 제거해 프로그램이 폭주하지 않도록 대처해야 한다.

`I/O 부하가 높은 경우`

I/O 부하가 높은 경우, 입출력이 많아서 부하가 높거나 스왑이 발생하여 디스크 액세스가 발생하는 경우가 대부분이다.\
sar 혹은 vmstat 으로 스왑의 상황을 확인해야 한다.

스왑이 발생할 경우
- 특정 프로세스가 극단적으로 메모리를 소비하고 있지 않은지 확인
- 프로그램 오류로 메모리를 지나치게 잡아먹는경우 프로그램 개선
- 메모리가 부족한 경우 증설또는 분산이 필요

디스크 액세스가 빈번한 경우 -> 캐시메모리가 부족한 경우
- 메모리 증설로 페이지 캐시 영역을 증설한다.
- 메모리 증설이 불가능 할 경우 데이터 분산 혹은 캐시서버 도입 등을 검토해야 한다.

