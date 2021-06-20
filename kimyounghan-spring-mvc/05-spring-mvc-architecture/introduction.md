# 스프링 MVC 시작하기

## SpringMemberFormControllerV1

### @RequestMapping

- 예전에는 스프링 MVC가 약해서 다른 프레임워크를 함께 사용했다.
- @RequestMapping 기반의 애너테이션 컨트롤러가 등장하면서 MVC의 완승으로 끝났다.

@RequestMapping을 위해 RequestMappingHandlerMapping, RequestMappingHAndlerAdapter를 사용한다.

{% tabs %} {% tab title="SpringMemberFormControllerV1.java" %}

```java

@Controller
public class SpringMemberFormControllerV1 {

    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        return new ModelAndView("new-form");
    }
}
```

{% endtab %} {% endtabs %}

- 요청 정보를 매핑한다.
- 해당 url이 호출되면 이 메서드가 호출된다.
- 에너테이션 기반으로 동작하기 때문에 메서드 이름은 임의로 지어도 된다.

### @Controller

![](../../.gitbook/assets/kimyounghan-spring-mvc/05/screenshot%202022-02-15%20오후%208.40.33.png)

- 스프링이 자동으로 스프링 빈으로 등록한다.
    - 내부에 @Component가 있어서 컴포넌트 스캔의 대상이 된다.
- 스프링 MVC에서 애너테이션 기반 컨트롤러로 인식한다.
    - @Controller가 있는 게 확인되면 RequestMappingHandlerMapping에서 이건 핸들러 정보구나 하고 꺼낼 수 있다는 의미다.

```java

@Component // 컴포넌트 스캔을 통해 스프링 빈으로 등록한다.
@RequestMapping // RequestMappingHandlerMapping이 이걸 보고 찾아낸다.
public class SpringMemberFormControllerV1 {
    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        return new ModelAndView("new-form");
    }
}
```

![](../../.gitbook/assets/kimyounghan-spring-mvc/05/screenshot%202022-02-15%20오후%208.48.53.png)

RequestMappingHandlerMapping은 스프링 빈 중에서 @RequestMapping 또는 @Controller가 **클래스 레벨**에 붙어있으면 매핑 정보로 인식한다.


### ModelAndView

- 모델과 뷰 정보를 담아 반환한다.

{% tabs %} {% tab title="SpringMemberFormControllerV1.java" %}

```java
@Controller
public class SpringMemberFormControllerV1 {

    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        // 이전에는 직접 만든 ModelView를 썼지만 이제는 스프링이 제공하는 ModelAndView를 쓴다.
        return new ModelAndView("new-form");
    }
}
```

{% endtab %} {% tab title="SpringMemberSaveControllerV1.java" %}

```java
@Controller
public class SpringMemberSaveControllerV1 {

    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")
    // paramMap 대신 서블릿 request, response를 받는다.
    public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView modelView = new ModelAndView("save-result");
        modelView.addObject("member", member);

        return modelView;
    }
}
```

{% endtab %} {% tab title="SpringMemberListControllerV1.java" %}

```java
@Controller
public class SpringMemberListControllerV1 {

  private final MemberRepository memberRepository = MemberRepository.getInstance();

  @RequestMapping("/springmvc/v1/members")
  public ModelAndView process() {
    List<Member> members = memberRepository.findAll();
    ModelAndView modelView = new ModelAndView("members");

    modelView.addObject("members", members);

    return modelView;
  }
}
```

{% endtab %} {% endtabs %}
