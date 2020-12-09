# 코틀린 과 스프링을 함께 사용시 주의

- 기존 레거시 프로젝트에서 @RequestScope 빈을 사용중인 것을 Kotlin 으로 옮기는 작업중 이슈가 발생함.
- 해당 빈이 제대로 생성되지 않고, 프록시가 동작하지 않음..
- @RequestScope 의 기본전략은 Target-Class (클래스기반 프록시)
- Spring Boot 에서는 기본적으로 Cglib 를 사용한다. (Class-Proxy 가능)
- 관련된 클래스는 아래 세가지이다.
  - AbstractRequestAttributesScope
  - AbstractAutowireCapableBeanFactory 
  - SimpleBeanTargetSource
- 기존 레거시 프로젝트에서는 프록시가 동작하고, 위 세가지 클래스를 사용해 빈을 새롭게 생성해서 잘 동작함.
- 코틀린 프로젝트는 기존에 all-open, kotlin-spring 플러그인을 사용중
- 혹시나 해서 빌드된 파일을 열어보니 해당 빈의 모든 필드가 final 이다...
  - 직접 @Component 와 같이 스프링 애노테이션이 적용된 클래스는 플러그인적용이 되지만, super class 에는 적용이 안된다.
- 프록시를 사용하는 경우 일일히 모든 필드에 open 을 사용할순 없으니 플러그인으로 해결하는 방법을 찾아봐야함..
