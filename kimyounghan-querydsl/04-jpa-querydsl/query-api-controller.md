# 조회 API 컨트롤러 개발

```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberJpaRepository memberJpaRepository;

    @GetMapping("/v1/members")
    public List<MemberTeamDto> searchMemberV1(MemberSearchCondition condition) {
        return memberJpaRepository.search(condition);
    }
}

```

```text
http://localhost:8080/v1/members?teamName=teamB&ageGoe=31&ageLoe=35
```