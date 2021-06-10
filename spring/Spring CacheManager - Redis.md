# Spring CacheManager Redis

## GenericJacksonSerializer
- JSON 형태로 변환시 사용한다.
- JACKSON 특성상 타읍 정보가 필요한데, 만약 타입 정보를 함께 저장하지 않는다면, LinkedHashMap 으로 변환되어 다음과 같은 에러를 자주 보게 된다..
    - java.util.LinkedHashMap cannot be cast to tutorial.Person with root cause blah..blah..blah...
- 이는 deSerialize 하는 코드를 보면 알 수 있다.

`GenericJacksonSerializer`

```java
/*
 * (non-Javadoc)
 * @see org.springframework.data.redis.serializer.RedisSerializer#deserialize(byte[])
 */
@Override
public Object deserialize(@Nullable byte[]source)throws SerializationException{
    return deserialize(source,Object.class);
}

@Nullable
public<T> T deserialize(@Nullable byte[]source,Class<T> type)throws SerializationException{

    Assert.notNull(type, "Deserialization type must not be null! Please provide Object.class to make use of Jackson2 default typing.");
    
    if(SerializationUtils.isEmpty(source)){
        return null;
    }
    
    try{
        return mapper.readValue(source,type);
    } catch(Exception ex) {
        throw new SerializationException("Could not read JSON: "+ex.getMessage(),ex);
    }
}
```
- ObjectMapper 를 이용해서 deserialize 를 시도하는데 Type 정보 전달시 Object 타입을 넘겨주게 된다.

## JdkSerializer
- Spring CacheManager - Redis 사용시 기본 Serializer 이다.
- Value 저장시 클래스 관련 정보가 함께 저장된다.
- 최근에 ClassLoader 정보가 없으면 serialize / deserialize 가 제대로 되지 않는 이슈가 있었지만, fix 되었다.
    - spring boot 기본설정시...
    - 만약 custom 한 CacheManager 를 사용할경우, 수동으로 Serializer 등록시 ClassLoader 정보를 함께 넘겨주어야 한다.

## 참고
- https://github.com/spring-projects/spring-boot/issues/11822
- https://hyperconnect.github.io/2019/10/28/jackson-serialize-for-global-caching.html