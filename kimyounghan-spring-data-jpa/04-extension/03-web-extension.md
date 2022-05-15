# Web 확장

## 도메인 클래스 컨버터

{% tabs %} {% tab title="MemberController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;

    // 도메인 클래스 컨버터 사용 전
    @GetMapping("/members/{id}")
    public String findMember(@PathVariable("id") Long id) {
        Member member = memberRepository.findById(id).get();
        return member.getUsername();
    }

    // 도메인 클래스 컨버터 사용 후
    @GetMapping("/members2/{id}")
    public String findMember2(@PathVariable("id") Member member) {
        // 바로 찾아준다.
        return member.getUsername();
    }

    @PostConstruct
    public void init() {
        memberRepository.save(new Member("userA"));
    }
}
```

{% endtab %} {% endtabs %}

- 도메인 클래스 컨버터가 중간에 동작해서 id로 회원 엔티티 객체를 반환한다.
    - 리파지토리를 사용해 엔티티를 찾는다.
- 이 방식을 사용하면 엔티티는 단순 조회용으로만 사용해야 한다.
    - 트랜잭션이 없는 범위에서 엔티티를 조회하므로 엔티티를 변경해도 DB에 반영되지 않는다.

## 페이징과 정렬

{% tabs %} {% tab title="MemberController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class MemberController {

  ...

    @GetMapping("/members")
    public Page<Member> list(Pageable pageable) {
        Page<Member> page = memberRepository.findAll(pageable);
        return page;
    }
}
```

```json
{
  "content": [
    {
      "createdDate": "2022-05-22T11:58:18.047317",
      "lastModifiedDate": "2022-05-22T11:58:18.047317",
      "createdBy": "dc7d4eee-f366-4bcf-ab64-47b87f084771",
      "lastModifiedBy": "dc7d4eee-f366-4bcf-ab64-47b87f084771",
      "id": 1,
      "username": "user0",
      "age": 0,
      "team": null
    },
    {
      "createdDate": "2022-05-22T11:58:18.078782",
      "lastModifiedDate": "2022-05-22T11:58:18.078782",
      "createdBy": "3aea9092-a323-452a-ad4c-a0d405842ba5",
      "lastModifiedBy": "3aea9092-a323-452a-ad4c-a0d405842ba5",
      "id": 2,
      "username": "user1",
      "age": 1,
      "team": null
    },
    ...

    {
      "createdDate": "2022-05-22T11:58:18.121901",
      "lastModifiedDate": "2022-05-22T11:58:18.121901",
      "createdBy": "51da627f-8564-48a1-bfdb-33b96ca6fd25",
      "lastModifiedBy": "51da627f-8564-48a1-bfdb-33b96ca6fd25",
      "id": 20,
      "username": "user19",
      "age": 19,
      "team": null
    }
  ],
  "pageable": {
    "sort": {
      "sorted": false,
      "unsorted": true,
      "empty": true
    },
    "pageNumber": 0,
    "pageSize": 20,
    "offset": 0,
    "paged": true,
    "unpaged": false
  },
  "totalPages": 5,
  "totalElements": 100,
  "last": false,
  "numberOfElements": 20,
  "first": true,
  "size": 20,
  "number": 0,
  "sort": {
    "sorted": false,
    "unsorted": true,
    "empty": true
  },
  "empty": false
}
```

- 어떤 쿼리든 Pageable을 파라미터로 넘기면 페이징을 할 수 있다.

```text
/members?page=0&size=3&sort=id,desc&sort=username,desc
```

- 요청 파라미터에 page, size, sort을 넘기면 자동으로 처리해준다.
    1. 컨트롤러를 바인딩 할 때 Pageable이 있는지 확인한다.
    2. Pageable 인터페이스가 org.springframework.data.domain.PageRequest 객체를 생성한다.
- page
    - 현재 페이지
    - 0부터 시작
- size
    - 한 페이지와 노출할 데이터 건수
- sort
    - 정렬 조건
    - asc는 생략 가능

### 기본값 변경

```yaml
# 기본 페이지 사이즈
spring.data.web.pageable.default-page-size=20
  # 최대 페이지 사이즈
spring.data.web.pageable.max-page-size=2000
```

### 개별 설정

```java

@RestController
@RequiredArgsConstructor
public class MemberController {
    
   ...

    @RequestMapping(value = "/members_page", method = RequestMethod.GET)
    public String list(
            @PageableDefault(
                    size = 12,
                    sort = "username",
                    direction = Sort.Direction.DESC)
            Pageable pageable) {
    ...

    }
}
```

- @PageableDefault 설정이 우선하여 적용된다.

### 접두사

```java

@RestController
@RequiredArgsConstructor
public class MemberController {
    
   ...

    @RequestMapping(value = "/members_page", method = RequestMethod.GET)
    public String list(
            @Qualifier("member") Pageable memberPageable,
            @Qualifier("order") Pageable orderPageable) {
    ...

    }
}
```

```text
/members?member_page=0&order_page=1
```

-
- 페이징 정보가 둘 이상이면 접두사로 구분한다.
- @Qualifier
    - value에 접두사 이름을 추가하면 요청에서 `접두사_xxx`를 구분한다.

### Page 내용을 DTO로 변환

- 엔티티를 API로 바로 노출하면 문제가 발생한다.
- 꼭 DTO로 변환해서 반환한다.
- Page는 map()을 지원해서 내부 데이터를 변환할 수 있다.

{% tabs %} {% tab title="MemberDto.java" %}

```java

@Data
public class MemberDto {
    private Long id;
    private String username;

    public MemberDto(Member m) {
        this.id = m.getId();
        this.username = m.getUsername();
    }
}
```

{% endtab %} {% tab title="MemberController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;

    @GetMapping("/members")
    public Page<MemberDto> listPageMap1(Pageable pageable) {
        Page<Member> page = memberRepository.findAll(pageable);
        Page<MemberDto> pageDto = page.map(MemberDto::new);
        return pageDto;
    }

    // Page.map() 최적화
    @GetMapping("/members")
    public Page<MemberDto> listPageMap2(Pageable pageable) {
        return memberRepository.findAll(pageable).map(MemberDto::new);
    }
}
```

{% endtab %} {% endtabs %}

### Page를 1부터 시작하기

- 스프링 데이터는 Page를 0부터 시작한다.
- 1부터 시작하려면?

**직접 클래스 생성**

- Pageable, Page를 파리미터와 응답 값으로 사용하지 않고 직접 클래스를 만들어서 처리한다.
- Pageable 구현체인 PageRequest를 직접 생성해서 리포지토리에 넘긴다.
- 물론 응답값도 Page 대신 직접 만들어 제공해야 한다.

**one-indexed-parameters 설정**

```yaml
spring.data.web.pageable.one-indexed-parameters=true
```

```text
http://localhost:8080/members?page=1
```

```json
{
  "content": [
    ...
  ],
  "pageable": {
    "offset": 0,
    "pageSize": 10,
    // 0 인덱스
    "pageNumber": 0
  },
  // 0 인덱스
  "number": 0,
  "empty": false
}
```

- 이 방법은 web에서 page 파라미터를 -1로 처리할 뿐이다.
- 따라서 응답값인 Page 에 모두 0 페이지 인덱스를 사용하는 한계가 있다.