# 기초부터 다지는 ElasticSearch 운영노하우

## 10장 색인 성능 최적화
- 정적 매핑
- _all field 비활성화
- refresh_interval 변경
- bulk API 활용하여 색인
- 그외

### 정적 매핑
- ES 는 매핑 정보를 바탕으로 문서를 색인하고 저장한다.
- 매핑 정보를 생성하는 방법 / 매핑 정보 생성에 따라 발생하는 성능차이를 살펴보자.

#### 매핑 정보를 생성하는 방법
1. 동적 매핑 (dynamic mapping)
2. 정적 매핑 (static mapping)

`매핑 방식의 장단점`

| 방식 | 장점 | 단점 |
| --- | --- | --- |
| 동적 매핑 | 미리 매핑정보를 생성하지 않아도 됨 | 불필요한 필드가 생성될 수 있음 |
| 정적 매핑 | 필요한 필드만 정의해서 사용할 수 있음 | 미리 매핑 정보를 생성해야 함 |

동적 매핑을 사용하면 불필요한 필드가 생성될 수 있고, 이는 불필요한 색인 작업을 유발한다.\
이로 인한 성능저하는 **문자열 타입 필드** 에서 크게 작용한다.

`문자열의 동적 매핑`

```shell
// 동적 매핑
curl -XPOST -H 'Content-Type: application/json' http://localhost:9200/string_index/_doc?pretty -d '
{
  "mytype": "String Data Type"
}
'

// 매핑 결과
curl http://localhost:9200/string_index/_mapping?pretty

{
  "string_index" : {
    "mappings" : {
      "properties" : {
        "mytype" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```
- 문자열 타입 필드의 동적 매핑 결과 Text, Keyword 두 가지 타입의 매핑이 생성된다.
- 추가적으로 **ignore_above** 속성으로 인해 256자가 넘으면 색인에 포함하지 않는다.

`문자열의 정적 매핑`

```shell
// 정적 매핑
curl -XPUT -H 'Content-Type: application/json' http://localhost:9200/keyword_index?pretty -d '
{
  "mappings": {
      "properties": {
        "mytype": {
          "type": "keyword"
        }
      }
  }
}
'

// 매핑 결과
curl http://localhost:9200/keyword_index/_mapping?pretty

{
  "keyword_index" : {
    "mappings" : {
      "properties" : {
        "mytype" : {
          "type" : "keyword"
        }
      }
    }
  }
}
```

##### 성능 비교

`Sample Data`

```shell
wget https://raw.githubusercontent.com/benjamin-btn/ES-SampleData/master/sample10-1.json
```

`동적 매핑`

```shell
// 색인 요청
time curl -XPOST -H 'Content-Type: application/x-ndjson' http://localhost:9200/dynamic/_doc/_bulk?pretty --data-binary @sample10-1.json

// 색인 결과
real	0m21.539s
user	0m0.034s
sys	    0m2.869s
```
- 총 21.539s 경과
- 동적 매핑은 bulk API 를 통한 색인으로, 최초 색인된 문서를 확인하여 동적 매핑을 우선적으로 생성한 뒤 전체 문서를 색인한다.

`정적 매핑`

```shell
// 기존 인덱스 삭제
curl -XDELETE -H 'Content-Type: application/json' http://localhost:9200/dynamic?pretty

// 매핑 정보 생성
curl -XPUT -H 'Content-Type: application/json' http://localhost:9200/static -d '
{
  "mappings": {
    "properties": {
      "title": {
        "type": "keyword"
      },
      "publisher": {
        "type": "keyword"
      },
      "ISBN": {
        "type": "keyword"
      },
      "release_date": {
        "type": "date",
        "format": "yyyy/MM/dd HH:mm:ss ||yyyy/MM/dd||epoch_millis"
      },
      "description": {
        "type": "text"
      }
    }
  }
}
'

// 색인 요청
time curl -XPOST -H 'Content-Type: application/x-ndjson' http://localhost:9200/static/_doc/_bulk?pretty --data-binary @sample10-1.json

// 색인 결과
real	0m13.743s
user	0m0.036s
sys	    0m2.878s
```
- 총 13.743s 경과
- 정적 매핑을 통해 타입을 지정해 놓으면 두번씩 색인되어야 할 작업이 1번만 수행하기 때문에 색인 성능이 크게 향상된다.

> 정적 매핑을 통한 성능 향상은 **문자열 필드** 에서 효과가 가장 크므로, 문자열 필드가 많으면 많을수록 정적 매핑으로 타입을 반드시 지정해 주도록 하자.

### _all 필드 비활성화
- _all 필드는 사용자가 색인한 문서의 모든 필드 값을 하나의 문자열 필드로 색인하는 필드이다.
- 각 필드의 **include_in_all** 옵션을 false 로 지정하면, 해당 필드를 제외 할 수 있다.
- 6.0 버전 이후부터는 deprecated 되었으며, 필드 복사가 필요하다면 **copy_to** 를 사용할 것을 권장한다.

`copy_to`
- 해당 필드의 값을 지정한 필드로 복사한다.
- keyword 타입의 필드에 copy_to 파라미터를 사용해서 text 타입을 지정하여 형태소 분석이 가능하다.
- 이를 이용해 _all 칼럼과 동일한 기능 구현이 가능하다.

`_all 필드 비활성화 하기`

```shell
curl -XPUT -H 'Content-Type: application/json' http://localhost:9200/my_index?pretty -d '
{
  "mappings": {
    "my_type": {
      "_all": {
        "enabled": false
      },
      "properties": {
        "content": {
          "type": "text"
        }
      }
    }
  }
}
'
```

### refresh_interval 변경하기
- ES 에서 색인되는 문서들은 바로 세그먼트에 저장되는것이 아니다.
- 메모리 버퍼에 일정기간 캐싱되었다가 일정한 주기마다 세그먼트 단위로 저장한다.
- 이런 작업들을 **refresh** 라고 하며, refresh_interval 으로 이 주기를 설정할 수 있다.
    - 기본 값은 1초
- 만약 실시간 검색 엔진으로 사용한다면, 기본 값인 1초로 사용해야 한다.
    - 주기를 늘릴 경우 사용자의 요청에 의해 색인된 데이터가 검색되지 않을 수 있다.
- 대용량의 로그를 수집하는 것이 목적이라면 refresh_interval 을 설정해서 색인 성능을 확보할 수 있다.

> 인덱스 마이그레이션과 같은 대량의 작업을 할경우 refresh_interval 을 비활성화 해 두었다가\
> 작업이 완료된 후 refresh_interval 을 재설정 하는 방식으로 진행하면 I/O 비용을 절약할 수 있다.

### bulk API
- bulk API 는 한 번에 대량의 문서를 색인/삭제/수정 할 수 있다.

| bulk API 동작 | 설명 |
| --- | --- |
| index | 문서 색인. 인덱스에 지정한 문서 아이디 있을 경우 업데이트 |
| create | 문서 색인. 지정한 문서 아이디가 없을 경우에만 색인 가능 |
| delete | 문서 삭제 |
| update | 문서 변경 |


`non-bulk`

```shell
// 1만건 샘플데이
wget https://raw.githubusercontent.com/benjamin-btn/ES-SampleData/master/nonbulk.sh

// 요청
time /bin/sh nonbulk.sh

// 결과
real 2m6.591s
user 0m33.225s
sys 0m40.399s
```

`bulk`

```shell
// 1만건 샘플 데이터
wget https://raw.githubusercontent.com/benjamin-btn/ES-SampleData/master/sample10-2.json

// 요청
time curl -XPOST -H 'Content-Type: application/x-ndjson' http://localhost:9200/bulk/_doc/_bulk?pretty --data-binary @sample10-2.json

// 결과
real	0m1.639s
user	0m0.011s
sys	0m0.255s
```

> bulk API 사용시 1초 정도 소요 된다.

### 그외
- 문서 색인시 PUT 메서드로 문서의 ID 를 지정하여 색인하는것이 아닌, POST 메서드로 문서의 ID 생성을 ES 로 위임 하는 방법
  - PUT 으로 ID 를 지정할 경우, 기존 문서가 있는지 확인하는 과정이 추가되지만, POST 를 사용하면 문서 확인 과정이 생략 되기 때문에 색인 성능이 향상된다.
- 레플리카 샤드를 0으로 지정 하는 방법
  - 프라이머리 샤드가 색인을 완료한 후 레플리카 샤드까지 복제가 완료된 후 클라이언트에게 응답을 반환한다.
  - 레플리카 샤드가 없다면 각 프라이머리 샤드로만 색인 한 뒤 색인 작업을 마무리 하기 때문에 색인 성능이 향상된다.

## 정리
- 동적 매핑은 불필요한 필드를 만들 수 있어 성능 저하를 일으킬 수 있다.
- 정적 매핑은 필드한 필드만을 사용하기 때문에 성능 향상에 도움이 된다.
- _all 필드는 모든 필드를 합친 필드를 추가 생성하기 때문에 성능 저하를 일으킬 수 있다.
  - 이는 6.0 부터 deprecated
  - 만약 같은 기능이 필요하다면 copy_to 를 사용할것
- refresh_interval 을 조절하여 I/O 비용을 줄일 수 있다.
- 대량의 문서에 대해 작업시 bulk API 를 사용하면 눈에 띌 정도의 색인 성능 향상을 꾀할 수 있다.
- 문서 id 지정이 필요 없다면, POST 메서드를 이용해 색인 성능을 향상할 수 있다.
- 레플리카 샤드가 반드시 필요한 상황이 아니라면 프라이머리 샤드만 운영하는 것도 성능 향상의 방법이 될 수 있다.
