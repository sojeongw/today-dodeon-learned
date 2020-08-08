# 회원 관리 예제 - 웹 MVC 개발

## 홈 화면 추가

```java
@Controller
public class HomeController {

  @GetMapping("/")
  public String home() {
    return "home";
  }
}
```

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
  <div>
    <h1>Hello Spring</h1>
    <p>회원 기능</p>
    <p>
      <a href="/members/new">회원 가입</a>
      <a href="/members">회원 목록</a>
    </p></div>
</div> <!-- /container --> </body>
</html>
```

원래 아무것도 없으면 `index.html` 을 불러오게 되어있는데 이걸 실행하면 `home.html` 이 불러와진다.

![](../../.gitbook/assets/kimyounghan-spring-introduction/05/_2021-01-27__11.58.36.png)

톰캣이 요청을 받았을 때 스프링 컨테이너에 우선 던지고 없으면 정적 파일을 볼러오기 때문에, 컨트롤러에서 먼저 받아서 `home.html` 로 이동하는 것이다.

## 회원 등록

```java
@Controller
public class HomeController {

  @GetMapping("/")
  public String home(){
    return "home";
  }

  @GetMapping("/members/new")
  public String createForm() {
    return "members/createMemberForm";
  }
}
```

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
  <form action="/members/new" method="post">
    <div class="form-group">
      <label for="name">이름</label>
      <input type="text" id="name" name="name" placeholder="이름을 입력하세요"></div>
    <button type="submit">등록</button>
  </form>
</div> <!-- /container --> </body>
</html>
```

![](../../.gitbook/assets/kimyounghan-spring-introduction/05/스크린샷%202021-03-05%20오전%2010.52.36.png)

`home.html` 에서 `회원 가입` 을 누르면 `/members/new` 로 이동하므로 이 경로를 받아줄 컨트롤러와 뷰를 만든다.

{% tabs %}
{% tab title="createMemberForm" %}
```html
...

// 등록 버튼을 누르면 post 방식으로 /members/new에 데이터를 전송한다.
<form action="/members/new" method="post">
<div class="form-group">
<label for="name">이름</label>
// input 태그의 name 항목의 값이 서버로 내려지는 key가 된다.
<input type="text" id="name" name="name" placeholder="이름을 입력하세요"></div>
<button type="submit">등록</button>
</form>

    ...
```
{% endtab %}
{% endtabs %}

회원 가입을 누르면 그냥 `/members/new` 이라는 URL로 이동하는데, 이게 곧 GET 방식이므로 GET으로 매핑된 `createForm` 컨트롤러로 가서 `members/createMemberForm` 에 있는 form을 출력한다.

{% tabs %}
{% tab title="MemberController" %}
```java
@Controller
public class MemberController {

  private final MemberService memberService;

  @Autowired
  public MemberController(MemberService memberService) {
    this.memberService = memberService;
  }

  @PostMapping("/members/new")
  public String create(MemberForm form) {
    Member member = new Member();
    member.setName(form.getName());

    memberService.join(member);

		// member를 생성한 뒤에는 홈으로 redirect 한다.
    return "redirect:/";
  }
}
```
{% endtab %}
{% tab title="MemberForm" %}
```java
public class MemberForm {
  private String name;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}
```
{% endtab %}
{% endtabs %}

`MemberController`에 이 요청을 받을 PostMapping 컨트롤러를 만든다. 그럼 스프링은 `MemberForm`에서 키로 내려온 `name`을 `setName`으로 세팅해준다. 이제 우리는 `getName`으로 데이터를 불러올 수 있다.

## 조회

```java
@Controller
public class MemberController {

	...

  @GetMapping("/members")
  public String list(Model model) {
    List<Member> members = memberService.findMembers();
    // 템플릿에서 members 라는 이름으로 꺼낼 수 있다.
    model.addAttribute("members", members);

    return "members/memberList";
  }
}
```

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
  <div>
    <table>
      <thead>
      <tr>
        <th>#</th>
        <th>이름</th>
      </tr>
      </thead>
      <tbody>
			<!-- members 라는 모델 안에서 꺼낸 뒤 각각의 값을 출력한다. -->
      <tr th:each="member : ${members}">
			<!-- 자바의 getId()와 getName()으로 불러온다. -->
        <td th:text="${member.id}"></td> 
        <td th:text="${member.name}"></td>
      </tr>
      </tbody>
    </table>
  </div>
</div> <!-- /container -->
</body>
</html>
```