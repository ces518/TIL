# 실전! Querydsl - Querydsl 설정과 검증

#### Querydsl 설정과 검증
- `build.gradle`
```groovy
plugins {
    id 'org.springframework.boot' version '2.2.5.RELEASE'
    id 'io.spring.dependency-management' version '1.0.9.RELEASE'
    // querydsl 플러그인 추가
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
    id 'java'
}



group = 'study'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

configurations {
    developmentOnly
    runtimeClasspath {
        extendsFrom developmentOnly
    }
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // querydsl 라이브러리 추가
    implementation 'com.querydsl:querydsl-jpa'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}

test {
    useJUnitPlatform()
}
//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
//querydsl 추가 끝
```

- QType이라는것을 뽑아낸뒤 QType을 이용해 쿼리를 한다.

`테스트 엔티티 생성`
```java
@Entity
@Getter @Setter
public class Hello {

    @Id @GeneratedValue
    private Long id;
}

```
- gradle -> Tasks -> other -> compileQuerydsl 테스크 실행
    - **build/generated/querydsl/study/querydsl/entity/QHello 파일이 생성됨**

`실행 결과`
```java
/**
 * QHello is a Querydsl query type for Hello
 */
@Generated("com.querydsl.codegen.EntitySerializer")
public class QHello extends EntityPathBase<Hello> {

    private static final long serialVersionUID = 1910216155L;

    public static final QHello hello = new QHello("hello");

    public final NumberPath<Long> id = createNumber("id", Long.class);

    public QHello(String variable) {
        super(Hello.class, forVariable(variable));
    }

    public QHello(Path<? extends Hello> path) {
        super(path.getType(), path.getMetadata());
    }

    public QHello(PathMetadata metadata) {
        super(Hello.class, metadata);
    }
}
```

> 기본적인 자바 컴파일 사이클 안에 complileQuerydsl 테스크가 포함된다.

`주의점`
- generated 된 QType 파일을 git 등을 활용해 버전관리를 하지말것
    - 버전에 따라 세부 내용이 변경될 수 있음 .gitignore등을 활용해서 제외

`검증 테스트`
```java
@SpringBootTest
@Transactional
class QuerydslApplicationTests {

    @Autowired EntityManager em;

    @Test
    void contextLoads() {
        Hello hello = new Hello();
        em.persist(hello);

        // 최신버전에서는 JPAQueryFactory를 사용할것을 권장
        JPAQueryFactory query = new JPAQueryFactory(em);
        QHello qHello = new QHello("h");

        // Querydsl 사용시 쿼리와 관련된 것은 QType을 사용
        Hello result = query
                .selectFrom(qHello)
                .fetchOne();

        // 조회한 엔티티가 동일한 엔티티인지 검증
        assertThat(result).isEqualTo(hello);
        assertThat(result.getId()).isEqualTo(hello.getId());
    }

}
```

- Querydsl 최신버전에서는 JPAQueryFactory를 사용하는것을 권장함
- Querydsl 사용시 쿼리와 관련된 것들은 모두 QType을 사용한다.
