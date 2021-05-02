# 회원 조회 API

```java

@RestController
@RequiredArgsConstructor
public class MemberApiController {

  private final MemberService memberService;

  @GetMapping("/api/v1/members")
  public List<Member> membersV1() {
    return memberService.findMembers();
  }
}
```

엔티티를 그대로 반환하면 엔티티 내용이 외부에 다 노출된다.

- 특정 필드에 `@JsonIgnore`를 넣을 수도 있지만 다양한 api가 엔티티를 사용하기 때문에 사용하면 안된다.
- 화면에 뿌리기 위한 프레젠테이션 로직이 엔티티에 추가 되었다.

## 엔티티 외부 노출

실무에서는 member 엔티티의 데이터가 필요한 API가 계속 증가하게 된다. 어떤 API는 name 필드가 필요하지만, 어떤 API는 name 필드가 필요없을 수 있다.
결론적으로 엔티티 대신에 API 스펙에 맞는 별도의 DTO를 노출해야 한다.

```java

@RestController
@RequiredArgsConstructor
public class MemberApiController {

  private final MemberService memberService;

  @GetMapping("/api/v1/members")
  public Result memberV2() {
    List<Member> members = memberService.findMembers();

    List<MemberDto> collect = members.stream().map(member -> new MemberDto(member.getName()))
        .collect(Collectors.toList());

    return new Result(collect);
  }

  @Data
  @AllArgsConstructor
  static class Result<T> {

    private T data;
  }

  @Data
  @AllArgsConstructor
  static class MemberDto {

    private String name;
  }
}
```

별도의 DTO를 만들어준다.

- 내가 노출할 데이터만 DTO에 노출한다.
- Result로 한 번 감싸줘서 향후 필요한 필드를 추가할 수 있게 한다.
- 감싸주지 않으면 바로 json 배열 타입으로 나가면서 수정에 대한 유연성이 떨어진다.

```java

@Data
@AllArgsConstructor
static class Result<T> {
  private T data;
  private Integer count;
}
```

만약 수정 사항이 생기면 이렇게 추가할 수 있다.