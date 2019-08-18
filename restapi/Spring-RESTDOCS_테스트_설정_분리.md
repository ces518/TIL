# REST API - Spring - REST DOCS 테스트 DB 설정 분리하기
- 애플리케이션 환경과, 테스트 환경의 DB를 분리하기

#### PostgreSQL
- Docker 환경에서 PostgreSQL 실행
    - 컨테이명은 rest
    - post 5432 bind
    - password를 pass로 demon 실행

```
docker run --name rest -p 5432:5432 -e POSTGRES_PASSWORD=pass -d postgres
```
- PostgreSQL 가 성공적으로 떳다면 bash로 접근하여 확인
    - \l: 데이터베이스 목록 확인
    - \dt: 테이블 목록 확인
```
docker exec -i -t rest bash
su - postgres
psql -d postgres
psql (11.3 (Debian 11.3-1.pgdg90+1))
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)

postgres=# \dt
Did not find any relations.
postgres=# 


```
- 드라이버 의존성 추가
```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

#### datasource 기본 전략
- 기존에는 h2의 scope을 지정하지 않았기 때문에 default scope인 compile scope을 사용함.
- spring boot 기본 전략에 따라 h2가 classpath에 존재하면 h2 datasource가 설정이 된다.

#### data-source 설정
- h2의 scope을 test로 변경하고 postgresql을 의존성으로 추가하였기때문에 datasource는 postgresql을 사용
- h2가 아닌 postgresql을 기본으로 사용하기때문에 data-source설정이 필요함

```properties
spring.datasource.username=postgres
spring.datasource.password=pass
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.driver-class-name=org.postgresql.Driver
```

#### hibernate 설정
- hibernate를 통하여 테이블을 자동 생성 및 sql logging 관련 설정
    - logging.level.org.hibernate.SQL=DEBUG: 실행되는 쿼리를 출력
    - logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE: 실제 parameter로 binding되는 값 출력
    - spring.jpa.properties.hibernate.format_sql=true: 실행되는 쿼리 formatting
```properties
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
spring.jpa.properties.hibernate.format_sql=true

logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

#### TEST 설정 분리
- 기본 application.properties 설정을 유지하고 필요한 부분만 override 
    - test/resource/application-test.properties 파일 생성
- @ActiveProfiles를 활용
```java
@RunWith(SpringRunner.class)
//@WebMvcTest
@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureRestDocs
@Import(RestDocsConfiguration.class)
@ActiveProfiles("test")
public class EventControllerTest {
```

- test.properties
```properties
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver

spring.datasource.hikari.jdbc-url=jdbc:h2:mem:testdb
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
```
