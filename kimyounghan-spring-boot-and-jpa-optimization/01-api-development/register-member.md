# 회원 등록 API

- 템플릿 뷰를 반환하는 컨트롤러와 api 방식의 컨트롤러의 패키지를 분리한다.
    - 둘은 예외 처리를 하는 방식이 다르다.
    - ex. api 방식은 json 형식으로 응답을 보낸다.

## saveMemberV1

{% tabs %} {% tab title="MemberApiController.java" %}

```java
// @RestController = @Controller + @ResponseBody
// @ResponseBody는 데이터를 json이나 xml로 보낼 때 사용한다.
@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    @PostMapping("/api/v1/members")
    // @RequestBody가 json 데이터를 객체로 바꿔준다.
    public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
        Long id = memberService.join(member);

        return new CreateMemberResponse(id);
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

{% endtab %} {% tab title="Member.java" %}

```java

@Entity
@Getter
@Setter
public class Member {

  ...

    // validation을 위한 애너테이션
    @NotEmpty
    private String name;

}

```

{% endtab %} {% endtabs %}

- @RestController
    - @Controller + @ResponseBody
    - @ResponseBody는 데이터를 json이나 xml로 보낼 때 사용한다.
- @RequestBody
    - json으로 온 바디를 Member 객체로 변환한다.
- @Valid
    - javax validation을 체크해준다.

### 문제점

- 프레젠테이션 레이어(컨트롤러)의 validation을 위한 로직이 엔티티에 들어가있다.
    - 어떤 API에서는 validation이 필요하지 않을 수 있다.
    - 엔티티의 필드명을 바꾸면 API 스펙 자체가 바뀌어버린다.
    - 수 많은 엔티티 필드 중에 무슨 값이 들어올지 모른다.
    - 실무에서는 한 엔티티에 대해 API가 다양하게 만들어진다.
        - 하나의 엔티티에 각각의 API를 위한 모든 요구사항을 담기 어렵다.

## saveMemberV2

```java

@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    @PostMapping("/api/v2/members")
    public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
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

- API 요청 스펙에 맞춰 별도의 DTO를 파라미터로 받는다.
    - Entity와 프레젠테이션 계층 로직을 분리할 수 있다.
    - Entity와 API 스펙을 명확하게 분리한다.
    - Entity가 변해도 API 스펙이 변하지 않는다.
    - 실무에서는 API 스펙에 Entity를 노출하면 안된다.