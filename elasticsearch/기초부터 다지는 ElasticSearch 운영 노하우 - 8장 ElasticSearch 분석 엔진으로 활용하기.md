# 기초부터 다지는 ElasticSearch 운영노하우

## 8장 분석 엔진으로 활용하기
- Elastic Stack 의 의미와 시스템 구성
- Filebeat, Logstash, Kibana 설치
- Kibana 를 통한 데이터 시각화
- Elastic Stack 의 가용성 확보를 위한 이중화 방법

### Elastic Stack
- **Elastic Stack** 은 로그를 수집, 가공하고 이를 바탕으로 분석하는데 사용하는 플랫폼
  - 이전에는 ELK 라고 불렸음
- Elastic Stack 은 로그를 전송하는 Filebeat, 로그를 파싱하는 Logstash, 파싱된 문서를 저장하는 Elastic Search, 데이터를 시각화 하는 Kibana 로 구성된다.

`ELK`

![ELK](./images/ELK.png)

`ELK with MQ`

![ELk with MQ](./images/ELKwithMQ.png)

`ES Stack Process`
1. Filebeat 가 지정된 위치에 존재하는 로그 파일을 읽어 Logstash 서버로 전송한다.
2. Logstash 는 Filebeat 로 부터 받은 파일을 파싱해서 특정한 포맷으로 가공해서 Elastic Search 에 전송한다.
3. Elastic Search 는 해당 문서를 인덱스에 적재한다. (데이터 저장소의 역할)
4. Kibana 는 Elastic Search 에 적재된 데이터를 시각화 한다.

### Filebeat 설치
- https://www.elastic.co/kr/downloads/past-releases/filebeat-7-7-1

`rpm 설치`
```shell
sudo rpm -ivh ./filebeat-7.7.1-x86_64.rpm
```

- rpm 으로 설치하면 /etc/filebeat 디렉터리에 각종 환경 설정파일이 생성된다.

| 파일 명 | 설명 |
| --- | --- |
| fields.yml | Filebeat 가 Logstash 를 통하지 않고 ES 에 직접 전송시 사용할 타입과 필드를 정의 |
| filebeat.reference.yml | filebeat.yml 파일에서 설장가능한 모든 설정들이 예시로 제공 |
| filebeat.yml | Filebeat 환경설정 파일 |
| modules.d | Filebeat 가 JSON 파싱까지 진행할 때 사용하는 환경설정을 저장하는 디렉터리 |

`filebeat.yml 예시`

```yaml
filebeat.inputs:
- type: log
  enable: true
  path: 
    - /usr/local/nginx/logs/access.log
output.logstash:
  hosts: ['logstashserver:5044']
```
- nginx 의 access log 를 수집 대상으로 지정하고, 로그스테시 서버로 전송하는 예제

> 파일빗의 로그는 /var/log/filebeat 디렉터리에 존재한다.

### Logstash 설치
- Filebeat 와 마찬가지로 rpm 을 통해서 설치가 가능하다.
- Logstash 는 자바가 설치되어 있어야 한다.
- 만약 자바가 설치되어 있지 않다면, could not find java; set JAVA_HOME ... 과 같은 에러가 발생한다.
- 설치가 완료되면 /etc/logstash 디렉터리가 생성되며 환경설정을 위한 파일이 생성 된다.

| 파일 명 | 설명 |
| --- | --- |
| logstash.yml | Logstash 와 관련된 설정을 할 수 있는 기본 설정파일 |
| conf.d | 파싱에 사용할 플러그인과 파싱 룰을 정의하는 설정파일이 존재하는 디렉터리 |
| jvm.options | Logstash 실행시 설정할 JVM 옵션들을 설정하는 파일 |
| log4j.properties | 로깅 관련 설정 파일 |

- Logstash 에서 제공하는 기본 설정 값은 특별히 수정할 곳이 없다.
- 하지만 성능을 끌어낼 필요가 있을 경우 worker 나 batch size 와 같은 값을 튜닝해서 사용한다.

`성능 튜닝을 위한 옵션`

| 파일 명 | 설명 |
| --- | --- |
| pipeline.workers | Logstash 가 파싱을 수행할 때 사용할 워커의 개수를 지정한다.<br/> 기본적으로 CPU 코어수로 설정되어 있으며, CPU Usage 를 높이고 더 많은 양을 처리하고 싶다면 CPU 코어 수 보다 많게 설정한다. |
| pipeline.batch.size | 하나의 워커가 파싱하기 위한 로그의 단위 <br/> 기본 값은 125 이며 로그를 125개씩 모아서 파싱한다. |
| pipeline.batch.delay | 하나의 워커가 파싱하기 위한 로그를 모으는 시간 <br/> batch.size 만큼 모이지 않더라도 해당 시간이 지나면 파싱한다. |

`nginx-logs.conf`

```shell
input {
  beats {
    port => "5044"
  }
}

filter {
  grok {
    match => { "message" => "${NUMBER:request_time:float}" }
    %{NUMBER:upstream_response_time:float} %{IPORHOST:clientip}
    (?:-|(%{WORD})) ${USER:ident} \[%{HTTPDATE:timestamp}\]
    "(?:${WORD:verb} ${NOTSPACE:request}(?: HTTP/${NUMBER:httpversion})?|${DATA:rawrequest})"
    %{NUMBER:respose} (?:%{NUMBER:bytes}|-)${QS:referrer} %{QS:agent} %{QS:forwarder} }
  }
}

output {
  file {
    path => "/var/log/logstash/output.log"
  }
}
```
- input
  - Filebeat 를 통해 입력 받는 포트 설정
- filter
  - Logstash 에서 제공하는 패턴 매치 방식중 GROK 패턴을 이용해서 로그 파일에 대한 파싱 룰 정의
- output
  - 파싱 결과를 파일로 출력

> Logstash 의 로그 파일 위치는 /var/log/logstash/logstash-plain.log

### 키바나
- 사용법 이므로 생략

### Elastic Stack 이중화

| 구성 요소 | 이중화 방법 |
| --- | --- |
| Filebeat | 로그 수집대상이 되는 서버에서 동작하기 때문에 이중화 구성이 필요 없다. |
| Logstash | 1. 로드밸런서를 사용 <br/> 2. Filebeat 서버에 환경설정 시 다수의 Logstash 를 지정 |
| Kibana | Active/Standby |

`Logstash 이중화시 장단점`

| 방법 | 장점 | 단점 |
| --- | --- | --- |
| LB | Logstash 서버 증설/축소가 자유로움 | 하드웨어 혹은 소프트웨어 LB 가 필요 |
| 리스팅 기반 | 별도의 하드웨어 혹은 소프트웨어 LB 가 필요 없음 | Logstash 서버 증설/축소시 Filebeat 서버 설정을 모두 변경 해야함 |

## 정리
- Elastic Stack 은 Filebeat, Logstash, ElasticSearch Kibana 이렇게 여러 개의 컴포넌트들을 조합해서 로그를 수집, 분석, 시각화 하는 시스템
- Filebeat 는 로그가 발생하는 애플리케이션서버에서 동작하며 Logstash 로 전달하는 역할
- Logstash 는 Filebeat 로 부터 받은 로그들을 파싱하여 JSON 형태로 변환한 다음 ElasticSearch 에 전달한다.
- ElasticSearch 는 해당 문서를 저장하는 역할
- Kibana 는 ElasticSearch 에 저장된 문서를 조회 및 시각화 하는 역할