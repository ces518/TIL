# Spring 5 in Action

## 3장 데이터로 작업하기

### JDBC 를 사용해서 데이터 읽고 쓰기
- 관계형 데이터베이스를 사용할 경우 가장 많이 사용하는 방법은 JDBC / JPA 이다.
- 스프링은 두 가지 방식을 모두 지원하며, 스프링을 사용하면 좀 더 쉽게 사용할 수 있다.
- 스프링의 JDBC 지원은 **JdbcTemplate** 을 기반으로 한다.

`JdbcTemplate 을 사용하지 않는 일반적인 JDBC 사용 예제`
```java
class Sample {
    public Ingredient findById(String id) {
        Connection conn = null;
        PreparedStatement stmt = null;
        ResultSet rs = null;
        
        try {
            conn = dataSource.getConnection();
            stmt = conn.prepareStatement("select id, name, type from Ingredient where id = ?");
            stmt.setString(1, id);
            rs = stmt.executeQuery();
            Ingredient ingredient = null;
            if (rs.next()) {
                ingredient = new Ingredient(
                        rs.getString("id"),
                        rs.getString("name"),
                        Ingredient.Type.valueOf(rs.getString("type"))
                );
            }
            return ingredient;
        } catch (SQLException e) {
            // ??
        } finally {
            if (rs != null) {
                try {
                    rs.close();
                } catch (SQLException e) {}
            }
            if (stmt != null) {
                try {
                    stmt.close();
                } catch (SQLException e) {}
            }
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {}
            }
        }
    }    
}
```

- 위 코드의 문제점은 자원을 획득하고, 자원을 회수하는 코드가 반복되고, 가독성을 해친다는 점이다.
- JdbcTemplate 은 이런 반복적인 코드를 줄여주고, 사용하기 편리한 인터페이스를 제공한다.

#### JdbcTemplate 이란 ?
- 스프링 프레임워크에서 지원하는 JDBC 를 활용한 템플릿 클래스
- org.springframework.jdbc.core 의 핵심 클래스
- JDBC 의 사용을 단순화하고, 일반적인 오류를 방지하는데 도움을 준다.
- https://www.tutorialspoint.com/springjdbc/springjdbc_jdbctemplate.htm
- https://www.baeldung.com/spring-jdbc-jdbctemplate

#### JdbcTemplate 사용하기
- JdbcTemplate 을 사용하기 위해서는 spring-boot-starter-jdbc 의존성을 추가해 주어아햔다.
- 또한 여기서는 데이터베이스로 **h2 database** 를 사용한다.

`H2 Database`
- h2database.com
- bin/h2.sh 혹은 bin/h2.bat 로 실행하면 된다.
- 자바기반으로 실행되기 때문에 자바가 설치되어 있어야한다.
- 웹콘솔을 제공한다.
- embedded mode / server mode 를 모두 제공한다.
- MySQL과 같은 데이터베이스를 사용해도 되지만, 설치가 번거로우며, 다양한 DB 방언을 지원하기때문에 개발 및 학습용으로 매우 좋다.
- 웹콘솔에서 최초 접근시 db 파일을 생성해 주어야한다.
- DB를 생성한뒤, 웹콘솔로 접근이 가능하다.
- 이후부터는 tcp 네트워크 모드로 접근이 가능하다.
- https://developerhive.tistory.com/34


```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
    <scope>runtime</scope>
</dependency>
```

#### Jdbc 리포지토리 정의
- 식자재 리포지토리는 아래의 연산을 수행할 수 있어야 한다.
1. 데이터베이스의 모든 식자제 데이터를 쿼리하여 Ingredient 객체 컬렉션에 넣어야 한다.
2. id 를 사용해 하나의 Ingredient 를 쿼리해야 한다.
3. Ingredient 객체를 데이터베이스에 저장해야 한다.

`IngredientRepository`
```java
public interface IngredientRepository {
    Iterable<Ingredient> findAll();
    Ingredient findById(String id);
    Ingredient save(Ingredient ingredient);
}
```

- 위에서 정의한 인터페이스를 JdbcTemplate 을 이용해 구현해보자.

`JdbcIngredientRepository`
```java
@Repository
@RequiredArgsConstructor
public class JdbcIngredientRepository implements IngredientRepository {

    private final JdbcTemplate jdbc;

    // Spring 4.3 이후 부터 생략가능
    // @Autowired
    // Lombok 애노테이션 으로 생성자 생성
//    public JdbcIngredientRepository(JdbcTemplate jdbc) {
//        this.jdbc = jdbc;
//    }

    @Override
    public Iterable<Ingredient> findAll() {
        return jdbc.query("select id, name, type from Ingredient", this::mapRowToIngredient);
    }

    @Override
    public Ingredient findById(String id) {
        return jdbc.queryForObject("select id, name, type from Ingredient where id = ?", this::mapRowToIngredient, id);
    }

    @Override
    public Ingredient save(Ingredient ingredient) {
        jdbc.update("insert into Ingredient (id, name, type) values (?, ?, ?)",
                ingredient.getId(),
                ingredient.getName(),
                ingredient.getType().toString());
        return ingredient;
    }

    private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException {
        return new Ingredient(
                rs.getString("id"),
                rs.getString("name"),
                Ingredient.Type.valueOf(rs.getString("type"))
        );
    }
}
```
- 위 코드를 살펴보면, 의존성 주입을 위해 **생성자 주입** 방식을 사용했다.
- 기본적으로 Spring 에서 의존성을 주입받으려면, **@Autowired** 애노테이션을 사용해야한다.
- 위 코드에선 @Autowired 가 생략되어 있다.
- Spring 4.3 이후 부터 생성자가 하나만 존재하고, 생성자 파라미터로 들어올 객체가 빈으로 등록되어 있다면, @Autowired 생략이 가능하다.
- 롬복의 @RequiredArgsConstructor 애노테이션을 사용하여 final 멤버들을 인자로 갖는 생성자를 생성할 수 있어 좀 더 간결해 진다.
- 기존의 JDBC 코드와 비교했을때, SQLException 을 처리하기위한 try-catch 구문들이 모두 사라진걸 알 수 있다.
- 핵심은 ? -> JdbcTemplate 클래스의 execute 메소드
- SQLException 을 DataAccessException 체크 예외를 언체크 예외로 변환하여, 불필요한 try-catch 구문들을 제거한다.
- 또한 이는 스프링의 철학 중 하나 **PSA** 와 관련이 있다.
    - 데이터 액세스 기술의 종류와 상관없이 일관된 예외가 발생하도록 만들어준다.
  
```java
@Nullable
public <T> T execute(ConnectionCallback<T> action) throws DataAccessException {
    Assert.notNull(action, "Callback object must not be null");
    Connection con = DataSourceUtils.getConnection(this.obtainDataSource());

    Object var10;
    try {
        Connection conToUse = this.createConnectionProxy(con);
        var10 = action.doInConnection(conToUse);
    } catch (SQLException var8) {
        String sql = getSql(action);
        DataSourceUtils.releaseConnection(con, this.getDataSource());
        con = null;
        throw this.translateException("ConnectionCallback", sql, var8);
    } finally {
        DataSourceUtils.releaseConnection(con, this.getDataSource());
    }

    return var10;
}

protected DataAccessException translateException(String task, @Nullable String sql, SQLException ex) {
    DataAccessException dae = this.getExceptionTranslator().translate(task, sql, ex);
    return (DataAccessException)(dae != null ? dae : new UncategorizedSQLException(task, sql, ex));
}
```

