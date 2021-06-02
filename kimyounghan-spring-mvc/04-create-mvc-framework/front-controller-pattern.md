# 프론트 컨트롤러 패턴

![](../../.gitbook/assets/kimyounghan-spring-mvc/04/screenshot%202021-06-21%20오후%208.20.42.png)

프론트 컨트롤러 패턴을 도입하기 이전에는 공통 로직을 앞에 깔고 그 뒤에 별도의 컨트롤러를 깔았다. 특정 입구 없이 아무나 다 들어올 수 있기 때문에 공통 로직을 여러 군데에 만들어야 했다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/04/screenshot%202021-06-21%20오후%208.20.48.png)

도입 후에는 공통 로직을 한 쪽에 몰고 컨트롤러가 각자 로직을 처리한다.

## 특징

- 프론트 컨트롤러 서블릿을 하나 두고 여기서 클라이언트 요청을 모두 받는다.
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출한다.
- 입구를 하나로 두고 공통 로직을 처리할 수 있다.
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 된다.
    - 프론트 컨트롤러가 직접 필요한 것을 호출해줄 것이기 때문에 다른 데에서는 서블릿이 필요없다.

## 스프링 MVC와 프론트 컨트롤러

- 스프링 웹 MVC의 핵심도 프론트 컨트롤러에 있다.
- 스프링 MVC의 DispatcherServlet이 프론트 컨트롤러 패턴으로 구현되어 있다.

## V1: 프론트 컨트롤러 도입

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

## V2: View 분리

```java
public class MemberSaveControllerV1 implements ControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) {

      ...

      String viewPath = "/WEB-INF/views/save-result.jsp";
      RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
      dispatcher.forward(request, response);
    }
}
```

모든 컨트롤러에서 뷰로 이동하는 부분에 중복이 있다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/04/screenshot%202021-07-18%20오후%204.14.15.png)

별도로 뷰를 처리하는 객체를 만들어보자.

{% tabs %} {% tab title="MyView.java" %}

```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

{% endtab %} {% tab title="ControllerV2.java" %}

```java
public interface ControllerV2 {
    MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

{% endtab %} {% endtabs %}

따로 뷰 처리 하는 클래스를 만들어서 최상위 컨트롤러에서 이 타입을 반환하게 한다.

{% tabs %} {% tab title="MemberFormControllerV2.java" %}

```java
public class MemberFormControllerV2 implements ControllerV2 {
    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        return new MyView("/WEB-INF/views/new-form.jsp");
    }
}
```

{% endtab %} {% tab title="MemberSaveControllerV2.java" %}

```java
public class MemberSaveControllerV2 implements ControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        request.setAttribute("member", member);
        return new MyView("/WEB-INF/views/save-result.jsp");
    }
}
```

{% endtab %} {% tab title="MemberListControllerV2.java" %}

```java
public class MemberListControllerV2 implements ControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();
        request.setAttribute("members", members);
        return new MyView("/WEB-INF/views/members.jsp");
    }
}
```

{% endtab %} {% endtabs %}

각 컨트롤러는 MyView 타입을 반환한다.

{% tabs %} {% tab title="FrontControllerServletV2.java" %}

```java
@WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {

    private Map<String, ControllerV2> controllerMap = new HashMap<>();

    public FrontControllerServletV2() {
        controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServletV2.service");

        String requestURI = request.getRequestURI();
        ControllerV2 controller = controllerMap.get(requestURI);

        if(controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyView view = controller.process(request, response);
        view.render(request, response);
    }
}
```

{% endtab %} {% endtabs %}

프론트 컨트롤러에서 생성된 뷰로 렌더한다.