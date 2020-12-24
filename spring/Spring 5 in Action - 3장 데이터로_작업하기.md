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

#### JdbcRepository 적용하기
- 기존의 DesignTacoController 에 JdbcRepository 적용하면 다음과 같다.

```java
@Slf4j
@Controller
@RequestMapping("/design")
@RequiredArgsConstructor
public class DesignTacoController {
  
    private final IngredientRepository ingredientRepository;
    
    @GetMapping
    public String showDesignForm(Model model) {
      List<Ingredient> ingredients = new ArrayList<>();
      ingredientRepository.findAll().forEach(ingredients::add);
    
      for (Ingredient.Type type : Ingredient.Type.values()) {
        model.addAttribute(type.toString().toLowerCase(), ingredients.stream().filter(x -> x.getType() == type).collect(Collectors.toList()));
      }
    
      model.addAttribute("taco", new Taco());
    
      return "design";
    }
    
    // ...
}
```

#### 스키마 정의하기
- 스프링 부트는 **schema.sql** 이라는 파일이 애플리케이션 classpath 루트에 존재하면 애플리케이션이 실행될때 schema.sql 파일을 실행한다.
- 또한 식자재 데이터를 미리 저장해야 하는데, 이 또한 data.sql 이라는 이름의 파일로 실행을 해준다.

`schema.sql`
```sql
create table if not exists Ingredient (
    id varchar(4) not null,
    name varchar(25) not null,
    type varchar(10) not null
);

create table if not exists Taco (
    id identity,
    name varchar(50) not null,
    createdAt timestamp not null
);

create table if not exists Taco_Ingredients (
    taco bigint not null,
    ingredient varchar(4) not null
);

alter table Taco_Ingredients add foreign key (taco) references Taco(id);
alter table Taco_Ingredients add foreign key (ingredient) references Ingredient(id);

create table if not exists Taco_Order (
    id identity,
    deliveryName varchar(50) not null,
    deliveryStreet varchar(50) not null,
    deliveryCity varchar(50) not null,
    deliveryState varchar(2) not null,
    deliveryZip varchar(10) not null,
    ccNumber varchar(16) not null,
    ccExpiration varchar(5) not null,
    ccCVV varchar(3) not null,
    placedAt timestamp not null
);

create table if not exists Taco_Order_Tacos (
    tacoOrder bigint not null,
    taco bigint not null
);

alter table Taco_Order_Tacos add foreign key (tacoOrder) references Taco_Order(id);
alter table Taco_Order_Tacos add foreign key (taco) references Taco(id);
```

`data.sql`
```sql
delete from Taco_Order_Tacos;
delete from Taco_Ingredients;
delete from Taco;
delete from Taco_Order;

delete from Ingredient;

insert into Ingredient (id, name, type) values ('FLTO', 'Flour Tortilla', 'WRAP');
insert into Ingredient (id, name, type) values ('COTO', 'Corn Tortilla', 'WRAP');
insert into Ingredient (id, name, type) values ('GRBF', 'Ground Beef', 'PROTEIN');
insert into Ingredient (id, name, type) values ('CARN', 'Carnitas', 'PROTEIN');
insert into Ingredient (id, name, type) values ('TMTO', 'Diced Tomatoes', 'VEGGIES');
insert into Ingredient (id, name, type) values ('LETC', 'Lettuce', 'VEGGIES');
insert into Ingredient (id, name, type) values ('CHED', 'Cheddar', 'CHEESE');
insert into Ingredient (id, name, type) values ('JACK', 'Monterrey Jack', 'CHEESE');
insert into Ingredient (id, name, type) values ('SLSA', 'Salsa', 'SAUCE');
insert into Ingredient (id, name, type) values ('SRCR', 'Sour Cream', 'SAUCE');
```

#### JdbcTemplate 을 사용한 데이터 저장
- JdbcTemplate 을 사용하여 데이터를 저장하는 방법은 두 가지 방법이 있다.
1. JdbcTemplate.update() 메소드
2. SimpleJdbcInsert 클래스 사용

- 1번의 경우 JdbcIngredientRepository 를 구현하면서 사용했지만, 이는 간단한 케이스의 경우 문제가 없지만 처리가 복잡해질수록 코드가 난잡해질 수 있다.
- TacoRepository 와 OrderRepository 를 구현하며 비교해 보자.

#### TacoRepository
- TacoRepository 는 Taco 객체를 받아 이를 저장하는 일을 수행한다.

`TacoRepository`
```java
public interface TacoRepository {
    Taco save(Taco design);
}
```

`JdbcTacoRepository`
```java
@Repository
@RequiredArgsConstructor
public class JdbcTacoRepository implements TacoRepository {

    private final JdbcTemplate jdbc;

    @Override
    public Taco save(Taco taco) {
        long tacoId = saveTacoInfo(taco);
        taco.setId(tacoId);
        for (Ingredient ingredient : taco.getIngredients()) {
            saveIngredientToTaco(ingredient, tacoId);
        }
        return taco;
    }

    private long saveTacoInfo(Taco taco) {
        taco.setCreatedAt(new Date());
        PreparedStatementCreator psc = new PreparedStatementCreatorFactory(
                "insert into Taco (name, createdAt) values (?, ?)",
                Types.VARCHAR, Types.TIMESTAMP
        ).newPreparedStatementCreator(
                List.of(taco.getName(), new Timestamp(taco.getCreatedAt().getTime()))
        );
        KeyHolder keyHolder = new GeneratedKeyHolder();
        jdbc.update(psc, keyHolder);
        return keyHolder.getKey().longValue();
    }

    private void saveIngredientToTaco(Ingredient ingredient, long tacoId) {
        jdbc.update("insert into Taco_Ingredients (taco, ingredient) values (?, ?)", tacoId, ingredient.getId());
    }
}
```
- TacoRepository 는 jdbcTemplate 의 update 메소드를 사용해, persist 를 수행한다.
- 특이점이라면, Taco 데이터를 Insert 후 생성되는 Key 값을 반환하기 위해 코드 레벨에서 KeyHolder 객체를 사용했다는 점이다.

#### OrderRepository
- OrderRepository 도 마찬가지로 Order 객체를 받아 저장하는 일을 수행한다.

`OrderRepository`
```java
public interface OrderRepository {
    Order save(Order order);
}
```

`JdbcOrderRepository`
```java
@Repository
public class JdbcOrderRepository implements OrderRepository {
    private final SimpleJdbcInsert orderInserter;
    private final SimpleJdbcInsert orderTacoInserter;
    private final ObjectMapper objectMapper;

    public JdbcOrderRepository(JdbcTemplate jdbc, ObjectMapper objectMapper) {
        this.orderInserter = new SimpleJdbcInsert(jdbc)
                .withTableName("Taco_Order")
                .usingColumns("id");
        this.orderTacoInserter = new SimpleJdbcInsert(jdbc)
                .withTableName("Taco_Order_Tacos");
        this.objectMapper = objectMapper;
    }

    @Override
    public Order save(Order order) {
        order.setPlacedAt(new Date());
        long orderId = saveOrderDetails(order);
        order.setId(orderId);
        List<Taco> tacos = order.getTacos();

        for (Taco taco : tacos) {
            saveTacoToOrder(taco, orderId);
        }

        return order;
    }

    private long saveOrderDetails(Order order) {
        @SuppressWarnings("unchecked")
        Map<String, Object> values = objectMapper.convertValue(order, Map.class);
        values.put("placedAt", order.getPlacedAt());

        return orderInserter.executeAndReturnKey(values)
                .longValue();
    }

    private void saveTacoToOrder(Taco taco, long orderId) {
        Map<String, Object> values = Map.of("taco", orderId, "taco", taco.getId());
        orderTacoInserter.executeAndReturnKey(values);
    }
}
```
- OrderRepository 는 SimpleJdbcInsert 객체를 사용해 persist 를 수행한다.
- TacoRepository 와 비교했을때 차이는, KeyHolder 객체를 **직접 사용하지 않았다** 는 점이다.
- 핵심은 ? -> executeInsertAndReturnKeyInternal 메소드
  - 내부에서 KeyHolder 객체를 사용해 키를 반환해 준다.

```java
private Number executeInsertAndReturnKeyInternal(List<?> values) {
    KeyHolder kh = this.executeInsertAndReturnKeyHolderInternal(values);
    if (kh.getKey() != null) {
        return kh.getKey();
    } else {
        throw new DataIntegrityViolationException("Unable to retrieve the generated key for the insert: " + this.getInsertString());
    }
}
```

#### TacoRepository 와 OrderRepository 비교
- 두 구현을 비교해 보았을때, Persist 처리를 하는 코드는 OrderRepository 가 좀 더 간결하다.
- 간단한 경우 JdbcTemplate 을 직접 사용해도 되지만, 처리가 복잡해 질수록 SimpleJdbcInsert 객체를 사용하는 것을 권장한다.

### Spring data JPA 를 사용하기
- Spring data 프로젝트는 여러 하위 프로젝트로 구성되는 프로젝트 이다.
- 대부분의 하위 프로젝트는 다양한 데이터베이스 유형을 사용한 데이터 퍼시스턴스에 중점을 둔다.
- 대표적인 하위 프로젝트는 다음과 같다.
  - Spring data JPA
  - Spring data MongoDB
  - Spring data Neo4
  - Spring data Redis
  - Spring data Cassandra
  
- spring data jpa 를 사용하기 위해 spring-boot-starter-jpa 의존성을 추가해야 한다.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
- 또한 jpa 를 사용하기 위해서는 기존 도메인 객체에 사전작업이 필요하다.

`Ingredient`
```java
@Data
@RequiredArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED, force = true)
@Entity
public class Ingredient {

    @Id
    private final String id;
    private final String name;

    @Enumerated(EnumType.STRING)
    private final Type type;

    public enum Type {
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }
}
```
- @Entity 애노테이션은, JPA 가 관리해야하는 엔티티 클래스임을 선언한다.
  - 쉽게 생각하면, ORM 의 O 대상을 선언하는 것과 동일 
  - 그렇다면 R 은 어떻게 선언해야 하는건가 ?..
  - **@Table** 애노테이션을 사용해서 특정 엔티티 클래스와 특정 테이블을 매핑 할 수 있는데, 이를 생략할 경우 기본적으로 엔티티 클래스명을 테이블 명으로 식별한다.
  - 엔티티 클래스명과 테이블명이 **일치하지 않을 경우** , @Table 애노테이션을 사용해서 매핑정보를 선언해주도록 하자.
- @Id 애노테이션은, 해당 필드가, 엔티티 클래스의 **식별자** 임을 선언한다.
  - 엔티티 클래스는 반드시 식별자가 존재해야 하며, 이는 매핑되는 테이블의 PK 와 매핑된다.
  - 이 식별자를 직접 생성해줄것인지, 혹은 생성 전략을 사용할것인지는 **@GeneratedValue** 애노테이션 과 관련이 있는데
  - 이를 생략할 경우, 식별자를 직접 생성해주어야 하며, 생략하지 않을 경우, 생성 전략에 따라 식별자가 생성된다.
- @Enumerated 애노테이션은, EnumType 과 매핑하기 위해 사용한다.
  - 기본적으로 생략을해도 동작을 하지만, 생략하지 않고, 선언을 한 이유는, Enum 타입의 값을 persist 하는 전략과 관련이 있다.
  - 애노테이션을 생략 혹은 선언을 하였으나 전략을 지정하지 않은경우, 기본전략으로 EnumType.ORDINAL 을 사용한다.
  - 이는 실제 데이터가 persist 될때 Enum 의 순서값 (0, 1, 2..) 를 사용하게 되는데 이는 매우 위험한 방법이다.
  - 만약 모종의 사유로 Enum 의 순서가 변경되거나, 중간에 새로운 값이 추가 될 경우 0, 1, 3, 2 방식이 아닌, 새로 갱신된 기준으로 다시 채번하기 때문에
  - 데이터 불일치 문제가 발생 한다.
  - 따라서 반드시 **EnumType.STRING** 전략을 사용해야 한다. (이는 순서가 아닌, Enum 의 이름을 persist 한다.)

- https://www.popit.kr/하이버네이트는-어떻게-자동-키-생성-전략을-결정하/
- https://jojoldu.tistory.com/295

`Taco`
```java
@Data
@Entity
public class Taco {

    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private Date createdAt;

    @NotNull
    @Size(min = 5, message = "Name must be at least 5 characters long")
    private String name;

    @ManyToMany
    @Size(min = 1, message = "You must choose at least 1 ingredient")
    private List<Ingredient> ingredients;

    @PrePersist
    void createdAt() {
        this.createdAt = new Date();
    }
}
```
- @ManyToMany 애노테이션은 연관관계 매핑정보를 제공하기 위한 애노테이션이다.
  - 이름에서도 알수 있듯이 다대다 매핑을 하기위해 사용되며, 실무에서는 사용하는것은 **지양** 해야 한다.
  - 다대다 매핑 전략을 사용할경우, 다대다 매핑정보를 저장하기위한, 관계테이블 (a.k.a 중간 테이블) 을 생성하는데
  - 이는 구현체가 생성한 테이블을 그대로 사용할수 밖에 없다, 비즈니스를 위한 컬럼을 추가하는 등 작업이 불가능하다.
  - 실무에서 다대다 매핑 보다는, 다대일 / 일대다 , 일대다 / 다대일 매핑 으로 변경하여, 관계테이블을 사용하지 않고 엔티티 클래스로 직접 정의해서 사용할 것을 권장한다.
  - 이를 **엔티티로 승격** 이라고 표현한다.
- @PrePersist 애노테이션은, 엔티티가 persist 되기 전, 수행되어야 하는 처리가 있다면, 해당 메소드 위에 선언하여 처리할 수 있다.

#### JPA Repository 선언
- Spring data JPA 를 사용하면, JDBC 버전에서 우리가 직접 구현했던 메소드들을 자동으로 구현해준다.
- 기존에 구현했던 Repository 들을 JpaRepository 로 변경해보자.

`IngredientJpaRepository`
```java
public interface IngredientJpaRepository extends CrudRepository<Ingredient, String> {
}
```

`TacoJpaRepository`
```java
public interface TacoJpaRepository extends CrudRepository<Taco, Long> {
}
```

`OrderJpaRepository`
```java
public interface OrderJpaRepository extends CrudRepository<Order, Long> {
}
```
- JDBC Repository 에서 구현했던 메소드들이, 선언부분 조차 사라졌다... 이는 **CrudRepository** 와 관련이 있다.
- CrudRepository 인터페이스를 살펴보면, 반복적으로 사용되는 여러 메소드들이 이미 선언되어 있고, 이 메소드들의 구현은 JPA 가 처리해 준다.
- 또 눈여겨 봐야할 점은 @Repository 애노테이션이 사라졌다는 점이다.
- CrudRepository 의 구현체인 SimpleJpaRepository 가 이미 @Repository 애노테이션을 가지고 있기 때문에, 선언하지 않아도 빈으로 등록된다.

`CrudRepository`
```java
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    <S extends T> S save(S var1);

    <S extends T> Iterable<S> saveAll(Iterable<S> var1);

    Optional<T> findById(ID var1);

    boolean existsById(ID var1);

    Iterable<T> findAll();

    Iterable<T> findAllById(Iterable<ID> var1);

    long count();

    void deleteById(ID var1);

    void delete(T var1);

    void deleteAll(Iterable<? extends T> var1);

    void deleteAll();
}
```

> 최근에는 CrudRepository 보단, 좀 더 기능이 확장된 **JpaRepository** 를 사용한다.

#### JPA Repository 활용
- Spring data JPA 를 사용하면, 기본제공 메소드외에도 필요한 특정 쿼리도 손쉽게 만들 수 있다.
- 메소드 명 기반으로 생성할 수 있으며, 이를 **NamedQuery** 라고 한다.
- 메소드 명을 기반으로 생성하는 것은 특정 규칙이 있다.
- find, read, get 은 하나 이상의 개체를 읽는 동의어 규칙이다.
- 특정 개체의 수를 쿼리하고 싶다면 count 를 사용할 수 있다.
- 예를 들어, Order 엔티티의 DeliveryZip 을 기반으로 조회하고 싶다면 다음과 같이 NamedQuery 를 정의할 수 있다.

```java
List<Order> findByDeliveryZip(String deliveryZip);
```

- https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference

### 정리
- 스프링의 JdbcTemplate 은 JDBC 작업을 손쉽게 할수 있도록 도와준다.
- 데이터 추가를 쉽게 하고 싶다면, SimpleJdbcInsert 를 사용하자.
- Spring data JPA 를 사용하면, 인터페이스를 작성하듯이 persistence 작업을 쉽게 할 수 있다.
- JPA 를 사용하려면 반드시 **공부** 하고 쓰자
