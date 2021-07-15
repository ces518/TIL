# Kotlin JacksonModule Issue
- kotlin 1.4.10, jacksonKotlinModule 2.11.3 기준 Issue 발생
- kotlin inline class 를 serialize 할때 property 명에 suffix 가 생기는 문제
- jackson-kotlin-module 2.12.1 기준 해소되었다.

## 이슈 상황

```kotlin
inline class Quantity(private val value: Int)
inline class Level(private val value: Int)

@GetMapping("inline-jackson-issue")
fun inlineClassJacksonIssue() = HelloJackson(level = Level(1), quantity = Quantity(1))

data class HelloJackson(
    val level: Level,
    val quantity: Quantity,
)
```


```json
{
    "level-PgIsiNg": 1,
    "quantity-eMvcqu8": 1
}
```
- 우리가 기대하는 응답은 property 명 그대로 노출되는 것인데, 불필요한 suffix 가 생기고 있다.
- KotlinNamesAnnotationIntrospector 클래스의 구현의 문제가 있었고, 2020.10.14 기준 해소되었음

`jacksonModule 2.12.1 으로 변경 후 결과`

```json
{
    "level": 1,
    "quantity": 1
}
```

## 관련 이슈
- https://github.com/FasterXML/jackson-module-kotlin/issues/356