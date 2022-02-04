# V4: 단순하고 실용적인 컨트롤러

V3는 잘 설계되었지만 항상 ModelView를 생성하고 반환해야 해서 번거롭다. 좋은 프레임워크는 아키텍처도 중요하지만 개발자가 쉽게 사용할 수 있어야 한다.

## 구조

![](../../.gitbook/assets/kimyounghan-spring-mvc/04/screenshot%202021-07-18%20오후%206.51.29.png)

기본 구조는 V3와 같지만 컨트롤러가 ModelView 대신 ViewName만 반환한다.

{% tabs %} {% tab title="ControllerV4.java" %}

```java
public interface ControllerV4 {
    /**
     *
     * @param paramMap
     * @param model
     * @return
     */
    String process(Map<String, String> paramMap, Map<String, Object> model);
}
```

{% endtab %} {% tab title="MemberFormControllerV4.java" %}

```java
public class MemberFormControllerV4 implements ControllerV4 {
    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        return "new-form";
    }
}
```

{% endtab %} {% tab title="MemberSaveControllerV4.java" %}

```java
public class MemberSaveControllerV4 implements ControllerV4 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.put("member", member);
        return "save-result";
    }
}
```

{% endtab %} {% tab title="MemberListControllerV4.java" %}

```java
public class MemberListControllerV4 implements ControllerV4 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        List<Member> members = memberRepository.findAll();
        model.put("members", members);
        return "members";
    }
}
```

{% endtab %} {% endtabs %}

컨트롤러는 ViewName만 반환한다.

{% tabs %} {% tab title="FrontControllerServletV4.java" %}

```java
@WebServlet(name = "frontControllerServletV4", urlPatterns = "/front-controller/V4/*")
public class FrontControllerServletV4 extends HttpServlet {

    ...

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
        ...

        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);
        MyView myView = viewResolver(viewName);

        // 이젠 직접 모델을 제공하므로 모델 뷰에서 모델을 꺼낼 필요가 없다.
        myView.render(model, request, response);
    }

    ...
}
```

{% endtab %} {% endtabs %}

프론트 컨트롤러는 컨트롤러에 paramMap과 model을 함께 넘긴다.