# CaseInsensitiveMap 

- org.apache.commons.collections.map 패키지에 속해있는 
- CaseInsensitiveMap 은 키 값의 대소문자를 가리지않는다.


- 예제
```java
import java.util.Map;
import org.apache.commons.collections4.map.CaseInsensitiveMap;

public class CaseInsensitiveMapSample {
    
    public static void main(String[] args) {
        
        Map map = new CaseInsensitiveMap(); 
        map.put("Key", "value");
        
        System.out.println(map.get("key")); // :~ value
        System.out.println(map.get("KEY")); // :~ value
        System.out.println(map.get("Key")); // :~ value
        
        map.put("KEY", "value Over Write");
        
        System.out.println(map.get("key")); // :~ value Over Write
        System.out.println(map.get("KEY")); // :~ value Over Write
        System.out.println(map.get("Key")); // :~ value Over Write
        
    }
}
```

- 결과
```
value
value
value Over Write
value Over Write
value Over Write
```

- ibatis 나 MyBatis 같은 SQL 매퍼를 사용하는 환경에서 
- 결과셋을 맵으로 담을 경우 키값이 컬럼명의 대문자로 지정된다.
- 대소문자를 구분하지않고 결과를 얻으려면 좀더 유용하다.
