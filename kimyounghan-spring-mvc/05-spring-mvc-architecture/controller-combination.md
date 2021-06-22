# 스프링 MVC 컨트롤러 통합

{% tabs %} {% tab title="SpringMemberControllerV2.java" %}

```java
@Controller
@RequestMapping("/springmvc/v2")
public class SpringMemberControllerV2 {

    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/members/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("new-form");
    }

    @RequestMapping("/members/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView modelView = new ModelAndView("save-result");
        modelView.addObject("member", member);

        return modelView;
    }

    @RequestMapping("/members")
    public ModelAndView members() {
        List<Member> members = memberRepository.findAll();
        ModelAndView modelView = new ModelAndView("members");

        modelView.addObject("members", members);

        return modelView;
    }
}
```

{% endtab %} {% endtabs %}

- @RequestMapping을 메서드 단위로 붙일 수 있다.
- url 중 공통되는 부분은 클래스 레벨에서 선언해 중복을 제거할 수 있다.