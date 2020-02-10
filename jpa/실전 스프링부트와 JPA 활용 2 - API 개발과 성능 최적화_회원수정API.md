# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_회원수정API

#### 회원 수정 API
```java
@PutMapping("/api/v2/members/{id}")
public UpdateMemberResponse updateMemberV2 (@PathVariable Long id,
                                            @RequestBody @Valid UpdateMemberRequest request) {
    memberService.update(id, request.getName());
    Member findMember = memberService.findOne(id);
    return new UpdateMemberResponse(findMember.getId(), findMember.getName());
}

@Data
@AllArgsConstructor
static class UpdateMemberResponse {
    private Long id;
    private String name;
}

@Data
static class UpdateMemberRequest {
    @NotEmpty
    private String name;
}

@Transactional
public void update(Long id, String name) {
    Member findMember = memberRepository.find(id);
    findMember.setName(name);
}
```

- Update 용 Request, Response DTO를 별도로 생성한다.
    - 등록과, 수정은 변경점이 서로 다르게 때문에 초기에는 같을지 몰라도 서로 다르게 변화하기 때문에 등록/수정 DTO는 분리하는 것을 추천한다.



#### Update 메소드에서 Member를 반환하지 않는 이유 ?
- 커맨드와 쿼리를 분리하라 원칙
- Update는 뭔가 변경을 하는 커맨드성 메소드이다.
- Update 메소드에서 Member를 반환해버리면 커맨드와 쿼리를 같이해버리는 꼴이 되어버린다.
    - 추후 유지보수에 좋지 않다.

#### HttpMethod PUT
- PUT
    - 멱등을 유지해야 한다.
    - 같은것을 여러번 호출해도 결과가 같아야 한다.
