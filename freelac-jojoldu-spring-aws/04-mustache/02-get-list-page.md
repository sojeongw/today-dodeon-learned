# 전체 조회 화면 만들기

{% tabs %}
{% tab title="PostsRepository.java" %}
```java
public interface PostsRepository extends JpaRepository<Posts, Long> {
    @Query("SELECT P FROM Posts p ORDER BY p.id DESC")
    List<Posts> findAllDesc();
}
```
{% endtab %}
{% endtabs %}

SpringDataJpa에서 제공하지 않는 쿼리는 `@Query`로 작성할 수 있다.

### querydsl

규모가 큰 프로젝트에서는 FK의 조인, 복잡한 조건 등 여러 이유 때문에 Entity 클래스만으로 처리하기 어려워 querydsl을 사용한다. 등록/수정.삭제는 SpringDataJpa를 사용하고 조회는 querydsl로 진행한다. 특히 querydsl은 아래와 같은 장점이 있다.

- 타입 안정성이 보장된다.
    - 단순 문자열이 아니라 메서드로 쿼리를 생성하기 때문에 오타나 존재하지 않는 칼럼을 쓰면 IDE에서 검출된다.
- 많은 회사에서 사용한다.
- 레퍼런스가 많다.

{% tabs %}
{% tab title="PostsService.java" %}
```java
@RequiredArgsConstructor
@Service
public class PostsService {

    private final PostsRepository postsRepository;

    ...

    @Transactional(readOnly = true)
    public List<PostsListResponseDto> findAllDesc() {
        return postsRepository.findAllDesc().stream().map(PostsListResponseDto::new).collect(Collectors.toList());
    }
}
```
{% endtab %}
{% tab title="PostsListResponseDto.java" %}
```java
@Getter
public class PostsListResponseDto {
    private Long id;
    private String title;
    private String author;
    private LocalDateTime modifiedDate;
    
    public PostsListResponseDto(Posts entity) {
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.author = entity.getAuthor();
        this.modifiedDate = entity.getModifiedDate();
    }
}
```
{% endtab %}
{% tab title="IndexController.java" %}
```java
@Controller
@RequiredArgsConstructor
public class IndexController {

    private final PostsService postsService;
    
    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());
        
        return "index";
    }
}
```
{% endtab %}
{% endtabs %}

### @Transactional(readOnly = true)

트랜잭션 범위는 유지하되 조회 기능만 남겨두어 조회 속도를 개선한다. 등록, 수정, 삭제 기능이 전혀 없는 서비스 메서드에서 사용하면 좋다.

### map(PostsListResponseDto::new)

이 코드는  `.map(posts -> new PostsListResponseDto(posts))`와 같은 의미다. 결과로 넘어온 `posts`를 `PostsListResponseDto`로 변환하는 람다식이다.

### Model

서버 템플릿 엔진에서 사용하는 객체를 저장할 수 있다. 여기서는 `postsService.findAllDesc()`로 가져온 결과를 posts로 index.mustache에 전달한다.

{% tabs %}
{% tab title="IndexController.java" %}
```java
@Controller
@RequiredArgsConstructor
public class IndexController {

    private final PostsService postsService;

    ...

    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto) {
        return postsService.update(id, requestDto);
    }
}
```
{% endtab %}
{% endtabs %}

(작성중)

{% tabs %}
{% tab title="IndexController.java" %}
```java
@Controller
@RequiredArgsConstructor
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());

        return "index";
    }

    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto) {
        return postsService.update(id, requestDto);
    }
    
    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model) {
        PostsRequestDto dto = postsService.findById(id);
        model.addAttribute("post", dto);
        
        return "posts-update";
    }
}
```
{% endtab %}
{% endtabs %}

수정한 화면을 연결할 controller를 구현한다.

(작성중)

{% tabs %}
{% tab title="PostsService.java" %}
```java
@RequiredArgsConstructor
@Service
public class PostsService {

    private final PostsRepository postsRepository;

   ...
    
    @Transactional
    public void delete(Long id) {
        Posts posts = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id = " + id));
        
        postsRepository.delete(posts);
    }
}
```
{% endtab %}
{% endtabs %}

존재하는 Posts인지 확인하기 위해 엔티티를 조회한 후 그대로 삭제한다.