# 회원 등록 API

controller와 api는 패키지를 분리한다. 현재 예제의 controller는 뷰를 처리하는 곳이고 api는 api를 다루는 곳이기 때문에 서로 예외 처리를 하는 방식이 다르기 때문이다.

{% tabs %} {% tab title="MemberApiController.java" %}

```java
// @Controller와 @ResponseBody를 합친 기능
// @ResponseBody는 데이터를 바로 json이나 xml로 보낼 때 사용하는 애너테이션이다.
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

위 코드엔 몇 가지 문제가 있다.

- 프레젠테이션 레이어에 있는 컨트롤러 즉, 화면에서 들어오는 validation을 위한 로직이 Entity에 들어가있다. 
- 어떤 API에서는 validation이 필요하지 않을 수 있다.
- Entity의 필드명을 바꾸면 API 스펙 자체가 바뀌어버린다.
- 수 많은 Entity 필드 중에 무슨 값이 들어올지 모른다.
- 실무에서는 한 Entity에 대해 API가 다양하게 만들어지는데, 하나의 Entity에 각각의 API를 위한 모든 요청 요구사항을 담기는 어렵다.

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

위처럼 API 요청 스펙에 맞춰 별도의 DTO를 파라미터로 받아야 한다.

- Entity와 프레젠테이션 계층 로직을 분리할 수 있다.
- Entity와 API 스펙을 명확하게 분리한다.
- Entity가 변해도 API 스펙이 변하지 않는다.
- 실무에서는 API 스펙에 Entity를 노출하면 안된다.