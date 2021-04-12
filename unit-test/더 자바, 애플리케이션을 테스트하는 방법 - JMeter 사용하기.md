# 더 자바, 애플리케이션을 테스트하는 방법

## JMeter 사용하기
- Thread Group
    - 유저 그룹
    - Number of Threads : (동시에 몇명이 요청을 보내는가)
    - Ramp-up period : 스레드를 생성할 시간 (초당 요청수)
    - Loop Count : Sampler 를 얼마나 반복할 것인가
- Sampler
    - 유저가 해야할 일
    - HTTP Sampler
        - 요청을 보낼 호스트 / 포트 / URI / 요청 본문 등을 설정
    - 샘플러를 순차적으로 등록하는 것도 가능
- Listener
    - View Results Tree
    - View Results in Table
    - Summary Report
    - Aggregate Report
    - Response Time Graph
    - Graph Results
- Assertion
    - 응답 코드 / 응답 본문 확인
- CLI
    - jmeter -n -t 설정파일 -l 리포트 파일
    
- BlazeMeter
    - Chrome 에서 하는 액션들을 녹화해서 JMeter 설정 파일로 생성가능함
    - 해당 파일을 읽어들여 Sampler 로 만들어서 성능 테스트가 가능해짐