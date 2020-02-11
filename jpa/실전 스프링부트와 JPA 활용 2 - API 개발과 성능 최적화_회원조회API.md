# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_회원조회API

#### 회원 조회 
```java
@GetMapping("/api/v1/members")
public List<Member> membersV1 () {
    return memberService.findMembers();
}
```
> 엔티티를 그대로 내보내게 되면 회원 엔티티의 모든 데이터가 노출된다.


#### @JsonIgnore
- SpringBoot는 기본적으로 Jackson 을 사용한다.
- @JsonIgnore 를 사용하면 @ResponseBody로 응답할때 대상에서 제외시킨다.

`문제점`
- @JsonIgnore 를 사용하면 1차적으론 해결이 되는듯 하지만..
- 다른 API를 만들때 문제가 된다.
- A 에서는 orders가 필요하고, B에서는 필요없다면 ? ..  문제가된다
- 엔티티가 수정되면 API 스펙이 변경된다. (매우 심각한 문제)
    - 엔티티를 수정하기가 힘들어진다.
- 유연성이 떨어진다.
    - 아래의 구조에서 count를 추가해야한다면 ?
        - JSON 구조가 깨져버림 
```json
[
    {
        "id": 1,
        "name: "hi"
    },
    {
        "id": 2,
        "name": "hi2"
    }
]
```

> 엔티티에 Presentation 계층을 위한 로직이 들어가선 안된다.

#### 해결방법
- 응답에 필요한 껍데기 Object를 생성사용하고, Dto를 이용해 필요한 데이터만 노출한다.
```java
@GetMapping("/api/v2/members")
public Result memberV2 () {
    List<Member> findMembers = memberService.findMembers();

    /* DTO로 변환 */
    List<MemberDto> memberDtos = findMembers.stream()
            .map(findMember -> new MemberDto(findMember.getName()))
            .collect(Collectors.toList());

    // 껍데기 형태로 한번 감싸주어야한다.
    // -> 유연성을 위함
    return new Result(memberDtos.size(), memberDtos);
}

@Data
@AllArgsConstructor
static class Result<T> {
    private int count;
    private T data;
}

/* API 스펙에 추가할 것들만 Dto에 추가하면 된다.*/
@Data
@AllArgsConstructor
static class MemberDto {
    private String name;
}
```

- API를 만들때는 파라메터를 받거나, 응답시에도 절대 엔티티를 그대로 사용하지 말것
- 무조건 Dto를 만들어 사용하라.
