# 실전 스프링부트와 JPA 활용 2 - API 개발과 성능 최적화_회원등록API

#### API 개발 기본
- POSTMAN 설치
    - getpostman.com

#### 회원등록 API
```java
@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    @PostMapping("/api/v1/members")
    public CreateMemberResponse saveMemberV1 (@RequestBody @Valid Member member) {
        // 엔티티를 그대로 사용해선 안된다
        // api 스펙을위한 DTO를 사용해야한다.
        // > 엔티티가 수정되면 api스펙이 변경된다.
        // > 엔티티를 그대로 사용했을때 발생한 장애 케이스가 매우 많다.
        // 엔티티를 api 파라메터로 절대 쓰지말것
        // 외부에 노출해서도 안된다.
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @PostMapping("/api/v2/members")
    public CreateMemberResponse saveMemberV2 (@RequestBody @Valid CreateMemberRequest request) {
        // DTO를 사용한 케이스
        // 엔티티가 수정되어도 api에 영향을 받지않는다.

        Member member = new Member();
        member.setName(request.getName());

        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberRequest {
        private String name;
    }

    @Data
    static class CreateMemberResponse {
        private Long id;

        public CreateMemberResponse(Long id) {
            this.id = id;
        }
    }
}
```

#### API 개발시 주의점
- 엔티티를 절대 그대로 사용해서는 안된다.
    - 엔티티가 수정되면 api 스펙이 변경되어 버린다.
    - 사이드이펙트가 어디까지 퍼질지 측정이 안된다.
- API 개발시에는 반드시 DTO를 사용하고, 엔티티를 외부에 노출하지 말것.
