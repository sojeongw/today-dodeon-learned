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

- Entity를 그대로 반환하면 Entity 내용이 외부에 다 노출된다.
- 화면에 뿌리기 위한 프레젠테이션 로직이 Entity에 추가되면 안된다.
- 특정 필드에 `@JsonIgnore`를 넣을 수도 있지만 다양한 api가 Entity를 사용하기 때문에 사용하면 안된다.
- Entity가 변경되면 API 스펙이 바뀌어버린다.

## Entity 외부 노출

- Entity 대신에 API 스펙에 맞는 별도의 DTO를 노출해야 한다.
    - 어떤 API는 name 필드가 필요하지만, 어떤 API는 name 필드가 필요 없을 수 있다.

```java

@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    @GetMapping("/api/v2/members")
    public Result memberV2() {
        List<Member> members = memberService.findMembers();

        List<MemberDto> collect = members.stream()
                .map(member -> new MemberDto(member.getName()))
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

- 내가 노출할 데이터만 별도의 DTO로 만든다.
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

- 수정 사항이 생기면 이렇게 추가할 수 있다.