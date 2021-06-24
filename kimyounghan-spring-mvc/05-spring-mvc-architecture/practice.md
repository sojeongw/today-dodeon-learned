# 실용적인 방식

{% tabs %} {% tab title="SpringMemberControllerV3.java" %}

```java

@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm() {
        return "new-form";
    }

    @PostMapping("/save")
    public String save(
            @RequestParam("username") String username,
            @RequestParam("age") int age,
            Model model
    ) {

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();

        model.addAttribute("members", members);
        return "members";
    }
}
```

{% endtab %} {% tab title="SpringMemberControllerV2.java" %}

```java

@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {

    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("new-form");
    }

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView modelView = new ModelAndView("save-result");
        modelView.addObject("member", member);

        return modelView;
    }

    @RequestMapping
    public ModelAndView members() {
        List<Member> members = memberRepository.findAll();
        ModelAndView modelView = new ModelAndView("members");

        modelView.addObject("members", members);

        return modelView;
    }
}
```

{% endtab %} {% endtabs %}

V2는 ModelAndView를 계속 중복으로 만들어줘야 해서 붎편했다. V3는 Model을 받아 처리하고 String을 반환한다. 또한, post든 get이든 http 메서드 상관없이 요청을 받는 방식에서 @GetMapping 등의 애너테이션을 사용하도록 개선했다.

### Model 파라미터

- Model을 파라미터로 받아서 사용할 수 있다.

### ViewName

- 뷰의 논리 이름을 직접 반환할 수 있다.

### @RequestParam

- 요청 파라미터를 받을 때 사용한다.
  - request.getParameter("username")과 유사하다.
- GET 쿼리 파라미터, POST Form 방식 모두 지원한다.

### @RequestMapping

- @GetMapping 등 Http 메서드로 구분할 수 있다.