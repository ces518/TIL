# 실전! 스프링 데이터 JPA - Auditing

#### Auditing
- 엔티티 생성, 변경시 변경한 사람과 시간을 추적하고 싶을때 사용
    - 등록일
    - 수정일
    - 등록자
    - 수정자

> 등록일, 수정일을 반드시 남겨야한다. 운영시에 반드시 필요함 !

#### 순수 JPA 사용
```java
// 속성을 상속받는 JPA에서 제공하는 상속
@MappedSuperclass
@Getter
public class JpaBaseEntity {

    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;

    /**
     * Persist 하기 이전에 콜백
     */
    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        this.createdDate = now;
        this.updatedDate = now; // updateDate에 null이 있다면 쿼리가 매우 지저분해진다.
    }

    @PreUpdate
    public void perUpdate() {
        LocalDateTime now = LocalDateTime.now();
        this.updatedDate = now;
    }
}
```

`@MappedSuperClass`
- JPA에서 제공하는 상속관계 애노테이션 중 하나
- 속성만 상속받는 경우 사용한다.

`JPA에서 제공하는 이벤트 애노테이션`
- @PrePersist, @PostPersist
    - Persist 전/후 에 콜백
- @PreUpdate, @PostUpdate
    - Update 전/후에 콜백

#### Spring data JPA 사용
```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseTimeEntity {

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}
/**
 * JpaBaseEntity와 동일하게 동작한다.
 */
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseEntity extends BaseTimeEntity {

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```

- @CreatedDate
    - Persist 시점에 날짜를 세팅 해준다.
- @LastModifiedDate
    - Update 시점에 날짜를 세팅 해준다.
- @CreatedBy
    - Persist 시점에 등록자를 세팅 해준다.
- @LastModifiedBy
    - Update 시점에 수정자를 세팅 해준다.

`Spring data JPA Auditing 사용시 주의점`
```java
@EnableJpaAuditing
@SpringBootApplication
// SpringBoot 이기 때문에 생략이 가능하다.
//@EnableJpaRepositories(basePackages = "study.datajpa.repository")
public class DataJpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(DataJpaApplication.class, args);
    }

    @Bean
    public AuditorAware<String> auditorProvider() {
        // AuditorAware의 getCurrentAware를 구현한다.
        // 여기서 리턴해주는 값을 @CreatedBy, @LastModifiedBy 의 값에 세팅해준다.
        return () -> Optional.of(UUID.randomUUID().toString());
    }
}    
```

- 반드시 Application 클래스에 **@EnableJpaAuditing** 애노테이션을 사용해야 동작한다.
    - 애노테이션이 없다면 동작하지 않음.

- @CreatedBy, @LastModifiedBy 사용시 **AuditorAware를 빈으로 등록해주어야한다.**
    - AuditorAware의 getCurrentAware를 구현해 주어야함
    - 해당 메소드에서 리턴해주는 값이 @CreatedBy, @LastModifiedBy 에 사용된다.

#### BaseEntity, BaseTimeEntity 분리한 이유 
- 테이블 마다 특성이 다르기 때문에 어떤 테이블에서는 날짜데이터만 필요, 어떤 테이블에서는 등록자 까지만 필요한 경우가 발생할 수 있기 때문에
- 한 클래스로 합쳐버린다면 유연한 대처가 불가능 하다.
- 유동적으로 사용이 가능하도록 두개의 클래스로 쪼개버린것
