# 더 자바, 애플리케이션을 테스트하는 방법

## ApacheBench
- 간단한 성능 테스트 정도라면 충분히 사용가능함.
- 리눅스 계열이라면 기본적으로 설치가 되어있음

```shell
ab -n 100 -c 10 http://localhost:8080/study
```
- -n : 전체 요청
- -c : concurrency (스레드 수)

## JMeter 소개
- 성능 측정 및 부하 테스트 기능을 제공하는 오픈소스 자바 애플리케이션
- https://jmeter.apache.org/
- 다양한 형태의 애플리케이션 테스트를 지원한다
    - 웹
    - SOAP / REST
    - FTP
    - 데이터베이스 (JDBC 사용)
    - Mail 등..
- CLI 지원
- 주요 개념
    - Thread Group
        - 스레드당 유저한명
        - 즉 유저 그룹
    - Sampler
        - 유저가 수행할 액션
    - Listener
        - 응답을 받았을때 처리할 일
        - 리포팅 등..
    - Configuration
        - Sampler / Listener 가 사용할 설정 값
        - 쿠키 혹은 헤더
    - Assertion
        - 응답이 성공적인지 검증 하는 방법
        - 응답코드 / 본문 내용 등

## 대체제
- Gatling
- nGrinder