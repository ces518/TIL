# JPA 기본편 - Hello JPA

#### H2
- http://www.h2database.com/
- 최고의 실습용 DB
- 1.5M 로 가볍다
- 웹용 쿼리툴 제공
- MySQL, Oracle 데이터베이스 시뮬레이션 가능
- 시퀀스, AUTO Increment 기능 지원

설치후 h2.sh를 실행하면 웹 콘솔창이 나타난다.
- H2Server로 사용한다.

#### 프로젝트 생성
- maven project 생성

`pom.xml`
```xml
<!--  JPA 하이버네이트  -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-entitymanager</artifactId>
    <version>5.3.10.Final</version>
</dependency>
<!--  H2 데이터베이스   -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
```

#### JPA 설정하기 - persistence.xml
- JPA 설정파일
- /META-INF/persistnece.xml
- persistnce-unit name 으로 이름 지정

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <persistence version="2.2"
        xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
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

#### 데이터베이스 방언
- dialect
- JPA 는 특정 데이터베이스에 종속되지 않도록 설계되어 있음
- 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다름
- 방언: SQL 표준은 지키지 않는 특정 데이터베이스 만의 고유한 기능 
- 하이버네이트는 40가지 이상의 데이터베이스 방언을 지원한다.
