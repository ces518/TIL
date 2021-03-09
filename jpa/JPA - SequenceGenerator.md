# JPA SequenceGenerator

@SequenceGenerator (JPA 에서 제공) 는 내부적으로 pooled Optimizer 를 사용한다.
(allocationSize 옵션 사용)

@GenericGenerator (Hibernate 에서 제공) 은 기본이 NoOp Optimizer 를 사용하고 이는 매번 1개씩 fetch 하는 방식

이런 optimizer 들을 이용해서, 최적화를 할 수 있는데...

Hibernate 에서 제공하는 @GenericGenerator 를 사용하면 optimizer 를 지정해서 최적화가 가능함...

## 참고
- https://dzone.com/articles/jpa-hibernates-legacy-and
- https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#identifiers-generators-table
- https://vladmihalcea.com/hibernate-hidden-gem-the-pooled-lo-optimizer/