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

- update할 때 사용했던 member 대신 새로 쿼리해서 member 값을 가져온다.
    - 혹은 update 했던 값의 id 정도는 가져와서 다시 쿼리한다.
    - 기존 member를 그대로 사용하면 update에서 커맨드와 쿼리가 같이 있는 모양이 되기 때문이다.
    - 커맨드와 쿼리를 분리하는 연습을 하면 유지보수가 쉬워진다.
- 커맨드
    - update를 하기 위한 변경성 메서드

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

- 변경 감지에 의해 영속 상태의 member는 @Transactional이 끝나는 시점에 flush() 하고 DB 트랜잭션을 커밋한다.

## 오류 정정

- 회원 정보를 부분 업데이트 하지만 PUT을 사용하고 있다.
    - PUT
        - 전체 업데이트
    - PATCH, POST
        - 부분 업데이트