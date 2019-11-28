# JPA 기본편 - Hello JPA 애플리케이션 개발

#### JPA 구동방식
- Persistence 클래스
    - persistence.xml
    - 1.설정정보 조회
    - 2.생성
    - 3.EntityManagerFactory
    - 4.EntityManager 생성하여 사용

#### JPA 실행해보기
```java
public class JpaMain {

    public static void main(String[] args) {
        // Persistence클래스가 EntityManagerFactory를 생성할때 unitName을 인자로 받는다.
        // Factory를 생성하는 순간 데이터베이스와의 연결도 완료된다.
        // 애플리케이션 로딩 시점에 딱 하나만 생성해야한다.
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");


        // EntityManagerFactory로 부터 EntityManager를 생성함
        // 한 트랙잭션 단위마다 entityManager를 생성해주어야한다.
        EntityManager entityManager = emf.createEntityManager();

        entityManager.persist(member);

        // 자원 종료
        entityManager.clear();

        // 애플리케이션이 종료될때 EntityManagerFactory를 종료해야한다.
        emf.close();
    }
}
```

Persistence 클래스가 EntityManagerFactory를 생성할때 unitName을 인자로 받는다.

unitName은 persistence.xml에 지정해둔 unitname이다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

<!--  unit name 은 보통 데이터베이스당 하나로 만든다.  -->
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```

EntityManagerFactory는 애플리케이션 로딩시점에 딱 하나만 생성해야 한다.

EntityManagerFactory로부터 EntityManager를 생성하여 데이터베이스와 관련된 작업을 수행할 수 있으며, 

#### 객체와 테이블을 생성하고 매핑하기
- @Entity: JPA가 관리할 객체
- @Id: 데이터베이스 PK와 매핑

```java
@Entity
public class Member {
    @Id
    private Long id;
    private String name;

}
```

```sql
create table Member {
    id bigint not null,
    name varchar(255),
    primary key (id)
};
```

#### 주의 할 점
- 엔티티 매니저 팩토리를 애플리케이션 전체에서 하나만 생성이 되어야 한다.
- 엔티티 매니저는 스레드간 공유 X (사용한후 버러여 한다.)
- JPA의 모든 데이터 변경은 트랜잭션 내에서 이루어 져야한다.


#### JPQL 소개
- 가장 단순한 조회 방법
- em.find();
- 나이가 18살 이상인 회원을 모두 검색하고 싶다면 ?

#### JPQL
- JPA를 사용하면 엔티티 객체 중심으로 개발을 하게 된다
- 문제는 검색 쿼리
- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색
- 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하다.
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다.
- DB를 대상으로 쿼리를 날리면 특정 벤더에 종속되어 버린다.
- 이를 해결하기위해 객체를 대상으로 할 수 있는 JPQL을 지원한다.
- 객체지향 쿼리 언어이다.
- GROUP BY HAVING .. 등등 모두 지원한다.

#### 실습소스코드
```Java
public class JpaMain {

    public static void main(String[] args) {
        // Persistence클래스가 EntityManagerFactory를 생성할때 unitName을 인자로 받는다.
        // Factory를 생성하는 순간 데이터베이스와의 연결도 완료된다.
        // 애플리케이션 로딩 시점에 딱 하나만 생성해야한다.
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");


        // EntityManagerFactory로 부터 EntityManager를 생성함
        // 한 트랙잭션 단위마다 entityManager를 생성해주어야한다.
        EntityManager entityManager = emf.createEntityManager();


        // JPA 에서 모든 데이터 변경작업은 반드시 트랜잭션 내에서 해야한다.
        EntityTransaction tx = entityManager.getTransaction();
        // 트랜잭션 시작
        tx.begin();

        // DB 작업
        try {
            // 등록하기

            /*
            Member member = new Member();
            member.setId(2L);
            member.setName("HELLO NAME2");
            entityManager.persist(member);
            */

            // 조회하기
            Member member = entityManager.find(Member.class, 1L);
            System.out.println(member.getId() + " " + member.getName());

            // 삭제하기
            /*
            entityManager.remove(member);
            */

            // 수정하기
            // update 쿼리가 자동적으로 날아간다
            member.setName("HELLO UPDATE");


            // JPQL 을 사용하여 쿼리를 생성해 사용할 수 있다.
            // Member 객체를 대상으로 쿼리를 한다.
            // 대상이 테이블이 아니라 객체이다.
            // 각 벤더의 방언에 맞춰 변형을 해준다.
            List<Member> findMembers = entityManager.createQuery("select m from Member m", Member.class)
                    .setFirstResult(1) // 페이징에 용이하다.
                    .setMaxResults(5)
                    .getResultList();

            for (Member findMember : findMembers) {
                System.out.println(findMember.getName());
            }

            // 트랜잭션 종료
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            // 자원 종료
            // 내부적으로 DB 커넥션을 가지고 있기 때문에 반드시 닫아주어야 한다.
            entityManager.clear();
        }

        // 애플리케이션이 종료될때 EntityManagerFactory를 종료해야한다.
        emf.close();
    }
}
```
