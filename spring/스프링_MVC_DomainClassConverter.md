# Spring MVC DomainClassConverter
- Spring data jpa 는 Spring MVC를 위한 DomainClassConverter를 제공한다.

- DomainClassConverter
    - Spring data jpa 가 제공하는 Repository를 사용하여 ID에 해당하는 엔티티를 읽어온다.

- 의존성 추가
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

- Entity 등록
```java
package me.june.springbootmvc;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import java.time.LocalDateTime;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-13
 * Time: 15:49
 **/
@Getter @Setter
@Entity
public class Event {

    @Id @GeneratedValue
    private Long id;

    private String name;

    private LocalDateTime starts;
}
```

- Repository
```java
package me.june.springbootmvc;

import org.springframework.data.jpa.repository.JpaRepository;

/**
 * Created by IntelliJ IDEA.
 * User: june
 * Date: 2019-07-14
 * Time: 21:40
 **/
public interface EventRepository extends JpaRepository<Event, Long> {
}
```

- 정리
    - GET /sample/1 으로 요청을 보내면 1이라는 ID에 해당하는 Entity를 Repository에서 찾아서 Conversion 을 해준다.
    - ID에 해당하는 Entity가 존재하지않는다면 null 로 가져온다.
    
