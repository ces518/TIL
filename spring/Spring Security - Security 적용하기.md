# Spring Security - Security 적용하기
- 스프링 부트를 활용해서 스프링 시큐리티를 적용한다.
- 의존성만 추가하여도 스프링부트가 제공하는 기본적인 스프링 시큐리티 자동설정이 적용된다.

#### 의존성
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

기존의 문제 2가지 해결이된다.
- 기존의 문제점
    - 인증 방법이 존재하지 않음
    - 인증 방법이 존재하지않아 현재 사용자가 누구인지 알 방법이 없다.

#### 부트가 제공하는 자동설정
- 기본 유저가 생성된다
    - 기본 username
        - user
    - 기본 password
        - console에 매번 새로운 password가 지정된다.
- 모든 요청이 인증을 필요로 하게 된다.

로그인을 하면 이전과 같이 NullPointerException이 발생하지않고 정상적으로 접근이 된다.

#### 해결된 문제
- 인증을 할 수 있다.
- 현재 사용자 정보를 알 수 있다.

#### 새로운 문제점
- 인증없이 접근가능한 URL을 설정할 수 없다.
- 유저 계정이 무조건 하나이다.
- 권한을 확인할 방법이 존재하지 않는다.
- user가 admin page에 접근이 가능하다.
- password가 log에 남는다 (매우 좋지 않음)
    - logfile이 탈취당하는 등 상황이 발생되었을때 문제 발생
    - 절대 민감한 정보는 log에 남겨서는 안된다.
