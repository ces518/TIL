# 기초부터 다지는 ElasticSearch 운영노하우

## 7장 클러스터 성능 모니터링과 최적화
- 클러스터 상태 확인 방법
- 클러스터를 구성하는 노드 상태 및 정보 확인 방법
- 인덱스와 샤드의 상태 및 정보 확인 방법
- 성능 지표 확인 방법
- 성능과 관련된 문제 해결 방법

### 클러스터 상태 확인
- ES 는 cat API 를 통해 클러스터, 노드, 샤드의 상태 등 다양한 정보를 확인할 수 있도록 인터페이스를 제공한다.
- _cat/health 는 클러스터의 상태를 확인할 수 있다.

`상태 확인 요청`
```shell
curl http://localhost:9200/_cat/health?v
```
- 상태 확인요청시 v 옵션 파라미터를 제공하는데, v 옵션 파라미터를 추가할 경우 해당 값들이 어떤 것을 의미하는지 헤더값이 함께 제공된다.

`요청 결과`
```shell
// v 옵션을 제거한 상태
1614675511 08:58:31 ncucu yellow 1 1 26 26 0 0 25 0 - 51.0%

// v 옵션을 추가한 상태
epoch      timestamp cluster status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1614675567 08:59:27  ncucu   yellow          1         1     26  26    0    0       25             0                  -                 51.0%
```
- 위 옵션 외에도 format, pretty 옵션을 제공한다.
- 각 필드가 나타내는 의미는 다음과 같다.

| 필드 명 | 설명 |
| --- | --- |
| epoch | API 호출 시간을 UNIX 로 표현 |
| timestamp | API 호출 시간을 타임스탬프로 표현 |
| cluster | 클러스터의 이름 |
| status | 클러스터의 상태, green/yellow/red |
| node.total | 클러스터를 구성하는 전체 노드의 수 |
| node.data | 클러스터를 구성하는 데이터 노드의 수 |
| shards | 클러스터에 존재하는 전체 샤드의 수 *샤드수가 너무 많으면 성능에 영향을 미친다. 때문에 모니터링 대상중 하나* |
| pri | 클러스터에 존재하는 프라이머리 샤드의 수 *모니터링 대상* |
| relo | 클러스터에 재배치중인 샤드의 수 *너무 많다면 색인/검색 성능이 떨어질 수 있음* |
| init | 클러스터에 초기화 되고 있는 샤드의 수 |
| unassign | 클러스터에 어떤 노드에도 배치되지 않은 샤드의 수 *0이 아니라면 클러스터 안정성에 문제가 발생할 수 있으므로 원인 파악 필요* |
| pending_tasks | 클러스터 유지.보수 작업 중 실행되지 못하고 대기중인 작업의 수 *0이 아니라면 클러스터 부하 혹은 특정 노드가 서비스 불능일 가능성이 있음* |
| max_task_wait_time | 위 작업이 실행되는데 소요된 최대 시간 *클러스터 부하 상황을 나타내는 지표로 활용* |
| active_shards_percent | 전체 샤드 중 정상 동작하는 샤드의 비율 |

`클러스터의 상태`

| 값 | 설명 |
| --- | --- |
| green | 모든 샤드가 정상 동작중 |
| yellow | 모든 프라이머리 샤드는 정상 동작중이지만 일부 레플리카 샤드가 비정상 동작중 |
| red | 일부 프라이머리 샤드 / 레플리카 샤드가 정상동작중이지 않음 |

### 노드의 상태 확인
- 노드의 상태를 확인하는 API 는 _cat/nodes

`상태 확인 요청`
```shell
curl http://localhost:9200/_cat/nodes?v
```

`요청 결과`
```shell
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           51          95   0    0.05    0.03     0.05 dilmrt    *      ncucu-1
```

| 필드 명 | 설명 |
| --- | --- |
| ip | 노드의 ip 주소 |
| heap.percent | 힙 메모리 사용률 *85% 이상을 계속 유지한다면 OOM 발생 할 수 있음* |
| ram.percent | 메모리 사용률 *노드가 사용 가능한 전체 메모리중 사용률, 대부분 90% 이상을 사용하는데 페이지 캐시로 사용되기 때문* |
| cpu | 노드의 cpu 사용률 |
| load_%m | 1분, 5분, 15 분의 평균 Load Average *값이 크다면 부하가 많이 발생 중이다. CPU 코어 개수에 따라 다르기 떄문에 CPU Usage 와 함께 살펴볼것* |
| node.role | 노드의 역할<br/> *d = data, m = master, i = ingest <br/> ex) di = data/ingest* |
| master | 마스터 노드를 표시 별표 (*) 로 표시됨 |
| name | 노드의 이름 |

> help 파라미터를 활용하면 위 예제 외에도 다양한 정보들을 확인할 수 있다.
> help 파라미터는, 모든 cat API 에서 사용 가능함.

### 인덱스 상태 확인
- 클러스터와 마찬가지로 인덱스의 상태도 green, yellow, red 세 가지로 나뉜다.
- 인덱스의 상태를 확인하는 API 는 _cat/indices

`상태 확인 요청`
```shell
curl http://localhost:9200/_cat/indices
```

`요청 결과`
```shell
helath status index uuid pri rep docs.count docs.deleted store.size pri.store.size
yellow open dev_ngram_analyzer  1FAlz9cPQRKRmlAhERtigw 1 1 0 0   208b   208b
yellow open test-1              Eke_IDPaS8yWfrrOjK0X0Q 3 1 0 0   624b   624b
yellow open teams               4kvmm9OQSNGyuhmO2PZS-A 1 1 3 0 14.4kb 14.4kb
yellow open dev_engram_analyzer _KGf8O06QBSUf8quTmzYQg 1 1 0 0   208b   208b
yellow open dev_stop_analyzer   3DnPgwpaT3-PAFOO6zdyEA 1 1 0 0   208b   208b
green  open users               EDpgqstLQ0usvk8OAFAPIg 1 0 2 0  4.8kb  4.8kb
yellow open movie_search        TFqCM2M9S8qQ1vEQJVlkhQ 1 1 0 0   208b   208b
yellow open new_users           r8lPnGiDSaSjeMM3qAWcSA 1 1 2 0  4.5kb  4.5kb
yellow open dev_af_analyzer     01EPWYUOT4exixvQwziCdg 1 1 0 0   208b   208b
yellow open dev_html_analyzer   XClOd0gxSRW_kLeYU1_DBg 1 1 0 0   208b   208b
yellow open tests               BitVjZ8SSoKx2UxAIr3JxA 1 1 0 0   208b   208b
yellow open books               UMCU_JcQQo21Q87qb4BtEg 1 1 1 0  3.5kb  3.5kb
yellow open orders              CpQ8vPaMSdyY4LMFxhg4KA 1 1 1 0  3.3kb  3.3kb
yellow open shard_index         QW7NV1vsSp6L5FrRGI8YpA 5 1 0 0    1kb    1kb
yellow open dev_analyzer        YyEyRfPCSKyzl5U0z7S-kA 5 1 1 0    4kb    4kb
yellow open pubsub              LDpqNEN1SyKGhru36fiXyg 1 1 0 0   208b   208b
```

| 필드 명 | 설명 |
| --- | --- |
| health | 인덱스의 상태, 개발 인덱스의 상태값도 조회가 가능하며, 하나라도 yellow 라면 클러스터의 상태도 yellow 가 된다. |
| open | 인덱스의 사용 여부 open/close |
| index | 인덱스의 이름 |
| uuid | 인덱스의 uuid |
| pri | 인덱스를 구성하고 있는 프라이머리 샤드의 수 |
| rep | 인덱스의 replicaiton 의 수 <br/> *값이 0인 경우 프라이머리 샤드가 장애가 발생하면 대체할 레플리카 샤드가 존재하지 않기 때문에 RED 상태에 빠진다.* |
| docs.count | 인덱스에 저장된 문서의 수 |
| docs.deleted | 인덱스에서 삭제된 문서의 수 |
| store.size | 인덱스가 차지중인 전체 용량 *프라이머리 와 레플리카를 모두 포함한다.* |
| pri.store.size | 인덱스의 프라이머리 샤드가 차지중인 전체 용량 |

### 샤드의 상태 확인
- 샤드의 상태를 확인하는 API 는 _cat/shards

`상태 확인 요청`
```shell
curl http://localhost:9200/_cat/shards?v
```

`요청 결과`
```shell
index               shard prirep state      docs  store ip        node
dev_analyzer        4     p      STARTED       1  3.2kb 127.0.0.1 ncucu-1
dev_analyzer        4     r      UNASSIGNED
dev_analyzer        2     p      STARTED       0   208b 127.0.0.1 ncucu-1
dev_analyzer        2     r      UNASSIGNED
dev_analyzer        3     p      STARTED       0   208b 127.0.0.1 ncucu-1
dev_analyzer        3     r      UNASSIGNED
dev_analyzer        1     p      STARTED       0   208b 127.0.0.1 ncucu-1
dev_analyzer        1     r      UNASSIGNED
dev_analyzer        0     p      STARTED       0   208b 127.0.0.1 ncucu-1
dev_analyzer        0     r      UNASSIGNED
movie_search        0     p      STARTED       0   208b 127.0.0.1 ncucu-1
movie_search        0     r      UNASSIGNED
books               0     p      STARTED       1  3.5kb 127.0.0.1 ncucu-1
books               0     r      UNASSIGNED
teams               0     p      STARTED       3 14.4kb 127.0.0.1 ncucu-1
teams               0     r      UNASSIGNED
new_users           0     p      STARTED       2  4.5kb 127.0.0.1 ncucu-1
new_users           0     r      UNASSIGNED
...
```

| 필드 명 | 설명 |
| --- | --- |
| index | 인덱스 명 |
| shard | 인덱스의 샤드 번호, 샤드 번호는 0부터 시작한다. |
| prirep | 프라이머리/레플리카 샤드 여부 <br/> p = 프라이머리, r = 레플리카 |
| state | 샤드의 상태 |
| docs | 샤드에 저장된 문서의 수 |
| store | 샤드의 크기 |
| ip | 샤드가 배치된 데이터 노드의 IP | 
| node | 샤드가 배치된 데이터 노드의 이름 |

`샤드의 상태`
| 값 | 설명 |
| STARTED | 정상 |
| INITIALIZING | 초기화 하는 상태 |
| RELOCATION | 샤드가 다른 노드로 이동중인 상태 |
| UNASSIGNED | 샤드가 어떤 노드에도 배치되지 않은 상태 |

> UNASSIGNED 상태의 샤드는 옵션 값을 통해 다양한 정보를 확인할 수 있다.

`UNASSIGNED 원인 분석`
```shell
curl -s 'http://localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason' | grep -i unassgined
```

`요청 결과`
```shell
dev_analyzer        4 p STARTED
dev_analyzer        4 r UNASSIGNED CLUSTER_RECOVERED
dev_analyzer        2 p STARTED
dev_analyzer        2 r UNASSIGNED CLUSTER_RECOVERED
dev_analyzer        3 p STARTED
dev_analyzer        3 r UNASSIGNED CLUSTER_RECOVERED
dev_analyzer        1 p STARTED
dev_analyzer        1 r UNASSIGNED CLUSTER_RECOVERED
dev_analyzer        0 p STARTED
dev_analyzer        0 r UNASSIGNED CLUSTER_RECOVERED
movie_search        0 p STARTED
movie_search        0 r UNASSIGNED CLUSTER_RECOVERED
books               0 p STARTED
books               0 r UNASSIGNED CLUSTER_RECOVERED
teams               0 p STARTED
teams               0 r UNASSIGNED CLUSTER_RECOVERED
new_users           0 p STARTED
new_users           0 r UNASSIGNED INDEX_CREATED
dev_engram_analyzer 0 p STARTED
dev_engram_analyzer 0 r UNASSIGNED CLUSTER_RECOVERED
dev_stop_analyzer   0 p STARTED
dev_stop_analyzer   0 r UNASSIGNED CLUSTER_RECOVERED
shard_index         4 p STARTED
shard_index         4 r UNASSIGNED INDEX_CREATED
...
```

> 샤드가 배치되지 않은 원인중 가장 흔한 경우는 INDEX_CREATED (인덱스 생성 된 후) / NODE_LEFT (노드에 문제가 생겨 클러스터에서 제외됨) 두 가지 이다.

### 클러스터 성능 지표 확인
- 클러스터 성능 지표 확인은 _cluster/stats API 를 통해 확인 가능

`성능 지표 확인`
```shell
curl http://localhost:9200/_cluster/stats?pretty
```

`요청 결과`
```shell
{
  "_nodes" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "cluster_name" : "ncucu",
  "cluster_uuid" : "f1iz_G1PSJ6TG-vLK35OHg",
  "timestamp" : 1614683079769,
  "status" : "yellow",
  "indices" : {
    "count" : 16,
    "shards" : {
      "total" : 26,
      "primaries" : 26,
      "replication" : 0.0,
      "index" : {
        "shards" : {
          "min" : 1,
          "max" : 5,
          "avg" : 1.625
        },
        "primaries" : {
          "min" : 1,
          "max" : 5,
          "avg" : 1.625
        },
        "replication" : {
          "min" : 0.0,
          "max" : 0.0,
          "avg" : 0.0
        }
      }
    },
    "docs" : {
      "count" : 10,
      "deleted" : 0
    },
    "store" : {
      "size_in_bytes" : 39021
    },
    "fielddata" : {
      "memory_size_in_bytes" : 0,
      "evictions" : 0
    },
    "query_cache" : {
      "memory_size_in_bytes" : 0,
      "total_count" : 0,
      "hit_count" : 0,
      "miss_count" : 0,
      "cache_size" : 0,
      "cache_count" : 0,
      "evictions" : 0
    },
    "completion" : {
      "size_in_bytes" : 0
    },
    "segments" : {
      "count" : 8,
      "memory_in_bytes" : 14560,
      "terms_memory_in_bytes" : 9152,
      "stored_fields_memory_in_bytes" : 3904,
      "term_vectors_memory_in_bytes" : 0,
      "norms_memory_in_bytes" : 896,
      "points_memory_in_bytes" : 0,
      "doc_values_memory_in_bytes" : 608,
      "index_writer_memory_in_bytes" : 0,
      "version_map_memory_in_bytes" : 0,
      "fixed_bit_set_memory_in_bytes" : 0,
      "max_unsafe_auto_id_timestamp" : -1,
      "file_sizes" : { }
    },
    "mappings" : {
      "field_types" : [
        {
          "name" : "date",
          "count" : 2,
          "index_count" : 2
        },
        {
          "name" : "integer",
          "count" : 5,
          "index_count" : 3
        },
        {
          "name" : "keyword",
          "count" : 35,
          "index_count" : 8
        },
        {
          "name" : "long",
          "count" : 2,
          "index_count" : 2
        },
        {
          "name" : "object",
          "count" : 7,
          "index_count" : 4
        },
        {
          "name" : "text",
          "count" : 18,
          "index_count" : 9
        }
      ]
    },
    "analysis" : {
      "char_filter_types" : [
        {
          "name" : "html_strip",
          "count" : 1,
          "index_count" : 1
        }
      ],
      "tokenizer_types" : [
        {
          "name" : "edge_ngram",
          "count" : 1,
          "index_count" : 1
        },
        {
          "name" : "ngram",
          "count" : 1,
          "index_count" : 1
        }
      ],
      "filter_types" : [
        {
          "name" : "stop",
          "count" : 2,
          "index_count" : 2
        }
      ],
      "analyzer_types" : [
        {
          "name" : "custom",
          "count" : 7,
          "index_count" : 6
        }
      ],
      "built_in_char_filters" : [ ],
      "built_in_tokenizers" : [
        {
          "name" : "keyword",
          "count" : 1,
          "index_count" : 1
        },
        {
          "name" : "standard",
          "count" : 4,
          "index_count" : 3
        }
      ],
      "built_in_filters" : [
        {
          "name" : "lowercase",
          "count" : 1,
          "index_count" : 1
        }
      ],
      "built_in_analyzers" : [
        {
          "name" : "standard",
          "count" : 5,
          "index_count" : 3
        }
      ]
    }
  },
  "nodes" : {
    "count" : {
      "total" : 1,
      "coordinating_only" : 0,
      "data" : 1,
      "ingest" : 1,
      "master" : 1,
      "ml" : 1,
      "remote_cluster_client" : 1,
      "transform" : 1,
      "voting_only" : 0
    },
    "versions" : [
      "7.7.1"
    ],
    "os" : {
      "available_processors" : 2,
      "allocated_processors" : 2,
      "names" : [
        {
          "name" : "Linux",
          "count" : 1
        }
      ],
      "pretty_names" : [
        {
          "pretty_name" : "CentOS Linux 7 (Core)",
          "count" : 1
        }
      ],
      "mem" : {
        "total_in_bytes" : 1927020544,
        "free_in_bytes" : 85975040,
        "used_in_bytes" : 1841045504,
        "free_percent" : 4,
        "used_percent" : 96
      }
    },
    "process" : {
      "cpu" : {
        "percent" : 0
      },
      "open_file_descriptors" : {
        "min" : 322,
        "max" : 322,
        "avg" : 322
      }
    },
    "jvm" : {
      "max_uptime_in_millis" : 817335312,
      "versions" : [
        {
          "version" : "14.0.1",
          "vm_name" : "OpenJDK 64-Bit Server VM",
          "vm_version" : "14.0.1+7",
          "vm_vendor" : "AdoptOpenJDK",
          "bundled_jdk" : true,
          "using_bundled_jdk" : true,
          "count" : 1
        }
      ],
      "mem" : {
        "heap_used_in_bytes" : 643553872,
        "heap_max_in_bytes" : 1073741824
      },
      "threads" : 50
    },
    "fs" : {
      "total_in_bytes" : 53675536384,
      "free_in_bytes" : 46831087616,
      "available_in_bytes" : 46831087616
    },
    "plugins" : [ ],
    "network_types" : {
      "transport_types" : {
        "security4" : 1
      },
      "http_types" : {
        "security4" : 1
      }
    },
    "discovery_types" : {
      "zen" : 1
    },
    "packaging_types" : [
      {
        "flavor" : "default",
        "type" : "tar",
        "count" : 1
      }
    ],
    "ingest" : {
      "number_of_pipelines" : 1,
      "processor_stats" : {
        "gsub" : {
          "count" : 0,
          "failed" : 0,
          "current" : 0,
          "time_in_millis" : 0
        },
        "script" : {
          "count" : 0,
          "failed" : 0,
          "current" : 0,
          "time_in_millis" : 0
        }
      }
    }
  }
}
```

| 필드 명 | 설명 |
| --- | --- |
| docs.count | 클러스터에 색인된 전체 문서 수 |
| docs.deleted | 클러스터에서 삭제된 문서의 수 |
| store.size_in_bytes | 저장중인 데이터 크기를 bytes |
| fielddata.memory_size_in_bytes | 필드 데이터 캐시의 크기 <br/> 필드 데이터는 문자열 필드에 대한 통계 작업시 필요한 데이터이다. <br/> 필드 데이터의 양이 많으면 힙 메모리를 많이 차지하기 때문에, 모니터링이 필요함 <br/> 노드들의 힙 메모리 사용량이 높다면 우선적으로 체크 |
| query_cache.memory_size_in_bytes | 쿼리 캐시의 크기 <br/> 모든 노드들은 쿼리 결과를 캐싱하고 있다. 모니터링 필요 |
| segments.count | 세그먼트의 수 |
| segments.memory_size_in_bytes | 세그먼트가 사용중인 메모리의 크기, forcemerge API 를 사용하면 줄어들 수 있다. |
| versions | 클러스터를 구성중인 노드들의 버전 |
| jvm.versions | 클러스터를 구성중인 노드들의 JVM 버전 |

