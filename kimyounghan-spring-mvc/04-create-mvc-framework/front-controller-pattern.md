# V1: 프론트 컨트롤러 도입

![](../../.gitbook/assets/kimyounghan-spring-mvc/04/screenshot%202021-06-30%20오후%208.52.51.png)

기존 코드를 최대한 유지하는 프론트 컨트롤러를 도입해본다.

```java
public interface ControllerV1 {
    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

서블릿과 비슷한 모양의 컨트롤러 인터페이스를 만든다.

인터페이스로 정의한 이유는 회원을 저장하고, 리스트를 조회하는 등 다양한 컨트롤러를 구현과 관계없이 일관성 있게 불러올 수 있기 때문이다.

{% tabs %} {% tab title="MemberFormControllerV1.java" %}

```java
public class MemberFormControllerV1 implements ControllerV1 {
    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}

```

{% endtab %} {% tab title="MemberSaveControllerV1.java" %}

```java
public class MemberSaveControllerV1 implements ControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}

```

{% endtab %} {% tab title="MemberListControllerV1.java" %}

```java
public class MemberListControllerV1 implements ControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();
        request.setAttribute("members", members);

        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}

```

{% endtab %} {% endtabs %}

인터페이스를 구현하는 컨트롤러를 만들고 기존 서블릿 로직을 그대로 복붙한다.

```java
// front-controller/v1 하위에 어떤 경로가 들어와도 일단 이 서블릿을 무조건 호출하게 된다.
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {

    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        // 프론트 컨트롤러가 생성될 때 값을 넣는다.
        controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServletV1.service");

        // 요청한 url을 받을 수 있다.
        String requestURI = request.getRequestURI();

        // 다형성을 이용해 Form, Save, List를 부모 타입으로 받는다.
        ControllerV1 controller = controllerMap.get(requestURI);

        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        controller.process(request, response);
    }
}
```

만약 `localhost:8080/front-controller/v1/members/new-form`을 호출했다면 일단 FrontControllerServletV1.service()를 거친다.
requestURI에는 `/front-controller/v1/members/new-form`가 들어가고 이 키를 가지고 MemberFormControllerV1이 호출된다.