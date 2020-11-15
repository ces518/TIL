# Kotlin in Action

## 11장 DSL 만들기
- DSL (Domain-Specific Language) 를 사용해 코틀린스러운 API 를 방법
- 전통 API 와 DSL API 의 차이
- 코틀린 DSL 설계는 코틀린 언어의 특성들을 활용한다.

#### API 에서 DSL 로
- 궁극적인 목표는 코드의 가독성과 유지보수성을 좋게 유지하는 것
- 개별 클래스에 집중하는것 만으로는 쉽지 않다.
- 대부분의 소스코드는 다른 클래스들과 상호작용 하는데, 상호작용은 클래스의 API 를 통해 일어난다.
- 모든 개발자는 API 잘 만들기 위해 노력해야 한다.

#### API 가 깔끔하다 ?
- 코드를 읽는 사람이 어떤 일이 일어날지 명확하게 이해할 수 있어야 한다.
- 코드가 간결하고, 불필요한 구문이나 번잡한 준비코드가 가능한 적어야 한다.

#### 코틀린이 간결한 구문을 지원하는 방법
| 일반 구문 | 간결한 구문 | 언어 특성 |
|---|---|---|
| StringUtils.capitalize(s) | s.capitalize() | 확장 함수 |
| 1.to("one") | 1 to "one" | 중위 호출 |
| set.add(2) | set += 2 | 연산자 오버로딩 |
| map.get("key") | map["key"] | get 메소드에 대한 관례 |
| file.use({ f -> f.read() }) | file.use { it.read() } | 람다를 괄호 밖으로 빼내는 관례 |
| sb.append("yes") | with (sb) { append("yes") } | 수신 객체 지정 람다 |
