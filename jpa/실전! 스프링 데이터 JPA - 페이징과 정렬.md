# 실전! 스프링 데이터 JPA - 페이징과 정렬

#### 페이징과 정렬

`요청 파라미터`
- /members?page=0&size=3&sort=id,desc&sort=username,desc
- page: 현재페이지 번호, **0부터 시작한다.**
- size: 한페이지에 노출할 데이터 건수 
    - 기본값은 size가 20개로 가져옴.
- sort: 정렬 조건을 정의한다. 정렬속성,정렬속성 ASC|DESC .., **정렬 방향을 변경허고 싶다면 sort 파라메터를 추가할것**

`기본 값 세팅`
- 글로벌 세팅
```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/datajpa
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        # show_sql: true
        format_sql: true
  data:
    web:
      pageable:
        default-page-size: 10
```
- spring.data.web.pageable... 의 속성을 변경해주면 글로벌로 설정이 된다.

- 로컬 설정
    - @PageableDefault(size = 5) 애노테이션을 사용

```java
@GetMapping("/members")
public Page<Member> list(@PageableDefault(size = 5) Pageable pageable) { 
    // Pageable 파라메터를 받을수 있도록 지원 (페이지에 관련된 정보)
    // 인터페이스로 받지만, 스프링부트가 구현체로 받게끔 해준다.
    return memberRepository.findAll(pageable);
}
```


`접두사`
- 한 페이지에 페이징을 하는것이 두가지 이상일 경우
- @Qualifier 에 접두사명 추가 (접두사명_xxx)
    - ex) member_page=0&order_page=1

```java
@GetMapping("/members")
public Page<Member> list(@PageableDefault(size = 5) Pageable pageable,
                            @Qualifier("member") Pageable memberPageable, // member_page 값 바인딩
                            @Qualifier("order") Pageable orderPageable) { // order_page 값 바인딩
    // Pageable 파라메터를 받을수 있도록 지원 (페이지에 관련된 정보)
    // 인터페이스로 받지만, 스프링부트가 구현체로 받게끔 해준다.
    return memberRepository.findAll(pageable);
}
```

#### Page내용을 DTO로 변환하기
- 엔티티를 그대로 노출해선 안된다.
    - 내부 설계를 모두 노출하기 때문에 규약 등이 의미가 없어 진다.
- API 스펙이 변경되기 때문에 API방식의 개발을 할 경우 매우 민감함

> API 개발 시 DTO로 반환하는것이 중요

`DTO로 반환`
```java
@GetMapping("/members")
public Page<MemberDto> list(@PageableDefault(size = 5) Pageable pageable,
                            @Qualifier("member") Pageable memberPageable,
                            @Qualifier("order") Pageable orderPageable) {
    // Pageable 파라메터를 받을수 있도록 지원 (페이지에 관련된 정보)
    // 인터페이스로 받지만, 스프링부트가 구현체로 받게끔 해준다.
    Page<Member> page = memberRepository.findAll(pageable);
    Page<MemberDto> map = page.map(member -> new MemberDto(member.getId(), member.getUsername(), null));
    return map;
}
```

#### Page를 1부터 시작하기
- Pageable을 새롭게 정의해서 사용

```java
// Page 타입은 그대로 사용해도 좋음
@GetMapping("/members")
public Page<MemberDto> list(@PageableDefault(size = 5) Pageable pageable,
                            @Qualifier("member") Pageable memberPageable,
                            @Qualifier("order") Pageable orderPageable) {
    Pageable request = PageRequest.of(1, 2);
    ...
}
```

- spring.data.web.pageable.one-indexed-parameters 옵션을 true로 지정해주면 된다.
    - 단 이방식은 한계가 있음
    - **1부터 페이지가 시작되도록 동작은 하지만, Page 객체 내부의 정보는 1부터 시작이 되지 않음..**
