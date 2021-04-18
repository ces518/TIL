# 더 자바, 애플리케이션을 테스트하는 방법

## Chaos Monkey 응답 지연

`Repository Watcher 활성화`

```yaml
# Repository 빈 들에게 카오스멍키 와쳐로 등록
chaos.monkey.watcher.repository=true
```
- @Service 로 등록된 빈들은 기본적으로 와쳐가 활성화 되어 있다.
- 주의할 점은 Spring AOP 프록시를 사용하기 때문에 런타임 중에 비활성화는 가능하지만, 비활성화 되어있던 것을 활성화 할 수는 없다.
- 활성화 및 비활성화를 런타임에 API 를 통해 할 수 있다.

`카오스 멍키 활성화`

```shell
curl -XPOST localhost:8080/actuator/chaosmonkey/enable
```
- 스프링 액츄에이터의 endpoint 를 사용해 활성화가 가능하다.

`카오스 멍키 상태 확인`

```shell
curl localhost:8080/actuator/chaosmonkey/status
```

`카오스 멍키 와쳐 상태 확인`

```shell
curl localhost:8080/actuator/chaosmonkey/watchers

{"controller":false,"restController":false,"service":true,"repository":true,"component":false}
```
- 위에 언급한 대로 서비스는 기본적으로 활성화 되어 있다.

`카오스 멍키 지연 공격 설정`

```shell
curl -XPOST -H 'Content-Type: application/json' localhost:8080/actuator/chaosmonkey/assaults -d '{ "level":3, "latencyRangeStart": 2000, "latencyRangeEnd": 5000, "latencyActive": true }'
```

`카오스 멍키 지연 공격 설정 확인`

```shell
curl localhost:8080/actuator/chaosmonkey/assaults

{"level":3,"latencyRangeStart":2000,"latencyRangeEnd":5000,"latencyActive":true,"exceptionsActive":false,"exception":{"type":null,"arguments":null},"killApplicationActive":false,"memoryActive":false,"memoryMillisecondsHoldFilledMemory":90000,"memoryMillisecondsWaitNextIncrease":1000,"memoryFillIncrementFraction":0.15,"memoryFillTargetFraction":0.25,"runtimeAssaultCronExpression":"OFF","watchedCustomServices":null}%
```

### 카오스멍키 커스터마이징
- https://codecentric.github.io/chaos-monkey-spring-boot/2.1.1/#_customize_watcher