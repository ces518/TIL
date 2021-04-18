# 더 자바, 애플리케이션을 테스트하는 방법

## Chaos Monkey 에러 발생

`에러 발생 재현`

```shell
curl -XPOST -H 'Content-Type: application/json' localhost:8080/actuator/chaosmonkey/assaults -d '
{
  "level": 3,
  "latencyActive": false,
  "exceptionsActive": true,
  "exception.type": "java.lang.RuntimeException"
}
'
```

> https://codecentric.github.io/chaos-monkey-spring-boot/2.1.1/#_examples