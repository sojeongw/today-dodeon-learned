# 회원 수정 API

```java
@RestController
@RequiredArgsConstructor
public class MemberApiController {

  @PutMapping("/api/v2/members/{id}")
  public UpdateMemberResponse updateMemberV2(@PathVariable("id") Long id,
      @RequestBody @Valid UpdateMemberRequest request) {

    memberService.update(id, request.getName());
    // 트랜잭션이 끝난 후 다시 커리해서 가져온다.
    Member member = memberService.findOne(id);

    return new UpdateMemberResponse(member.getId(), member.getName());
  }

  @Data
  static class UpdateMemberRequest {
    private String name;
  }

  @Data
  @AllArgsConstructor
  static class UpdateMemberResponse {
    private Long id;
    private String name;
  }
}
```

update에서 member를 리턴받지 않고 새로 쿼리해서 가져온다.

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {
  
  @Transactional
  public void update(Long id, String name) {
    Member member = memberRepository.findOne(id);
    // 변경 감지를 사용한다.
    member.setName(name);
  }
}
```

조회한 member를 반환해도 되지만 커맨드와 쿼리를 분리하기 위해 하지 않았다. 커맨드는 update를 하기위한 변경성 메서드다. 하지만 member를 리턴하면 쿼리와 커맨드가 같이 있게 된다.

## 오류 정정

회원 수정 API updateMemberV2 은 회원 정보를 부분 업데이트 한다. 여기서 PUT 방식을 사용했는데, PUT은 전체 업데이트를 할 때 사용하는 것이 맞다. 부분 업데이트를 하려면 PATCH를 사용하거나 POST를 사용하는 것이 REST 스타일에 맞다.