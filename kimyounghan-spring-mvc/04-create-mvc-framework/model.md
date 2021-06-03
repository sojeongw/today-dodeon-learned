# V3: Model 추가

## 서블릿 종속성 제거

```java
public class MemberFormControllerV2 implements ControllerV2 {
    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        return new MyView("/WEB-INF/views/new-form.jsp");
    }
}
```

컨트롤러는 HttpServletRequest/Response가 꼭 필요하지 않다. 요청 파라미터 정보는 Map으로 넘기면 이 서블릿 기술을 몰라도 동작할 수 있다. Model은 HttpServletRequest를 사용하는 대신, 별도의 Model 객체를 만들어 반환하면 된다.

이렇게 하면 구현 코드가 단순해지고 테스트 코드 작성이 쉬워진다.

## 뷰 이름 중복 제거

```text
/WEB-INF/views/new-form.jsp -> new-form
```

일일이 뷰 이름을 MyView에 집어넣는 대신 논리 이름을 반환하도록 한다. 경로가 사라지게 되므로 뷰의 폴더 위치가 바뀌어도 프론트 컨트롤러만 고치면 된다.

## 구조

![](../../.gitbook/assets/kimyounghan-spring-mvc/04/screenshot%202021-07-18%20오후%206.09.33.png)

컨트롤러를 호출해서 모델을 받으면 뷰 리졸버에서 논리 이름을 받아 진행하는 형식이다.

## ModelView

지금까지는 서블릿에 종속적인 HttpServletRequest.setAttribute()를 통해 데이터를 저장하는 모델을 사용했다.

이제는 서블릿의 종속성을 제거하고 직접 Model을 만들고 View 이름까지 전달하는 객체를 만들어본다.

{% tabs %} {% tab title="ModelView.java" %}

```java
@Getter @Setter
public class ModelView {
    private String viewName;
    private Map<String, Object> model = new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }
}
```

{% endtab %} {% tab title="ControllerV3.java" %}

```java
public interface ControllerV3 {
    ModelView process(Map<String, String> paramMap);
}
```

{% endtab %} {% endtabs %}

- ModelView는 View 이름과 View를 렌더링할 때 필요한 Model 개체를 가진다. 
- 프론트 컨트롤러는 서블릿 기술 없이 paramMap에 HttpServletRequest의 파라미터 데이터를 담아 컨트롤러를 호출한다.
- 컨트롤러는 응답으로 ModelView 객체를 반환한다.

{% tabs %} {% tab title="MemberFormControllerV3.java" %}

```java
public class MemberFormControllerV3 implements ControllerV3 {
    @Override
    public ModelView process(Map<String, String> paramMap) {
        return new ModelView("new-form");
    }
}
```

{% endtab %} {% tab title="MemberSaveControllerV3.java" %}

```java
public class MemberSaveControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        // View 이름 추가
        ModelView modelView = new ModelView("save-result");
        // Model 데이터 추가
        modelView.getModel().put("member", member);

        return modelView;
    }
}
```

{% endtab %} {% tab title="MemberListControllerV3.java" %}

```java
public class MemberListControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        List<Member> members = memberRepository.findAll();
        
        // View 이름 추가
        ModelView modelView = new ModelView("members");
        // Model 데이터 추가
        modelView.getModel().put("members", members);

        return modelView;
    }
}
```

{% endtab %} {% endtabs %}

각 컨트롤러는 모델을 만들어 반환한다.

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

    // 모델을 다루는 render 메서드를 구현한다.
    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        // 모델에 있는 값을 다 꺼내서 HttpServletRequest의 attribute로 세팅한다.
        model.forEach(request::setAttribute);
    }
}
```

{% endtab %} {% tab title="FrontControllerServletV3.java" %}

```java
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/V3/*")
public class FrontControllerServletV3 extends HttpServlet {

    private Map<String, ControllerV3> controllerMap = new HashMap<>();

    public FrontControllerServletV3() {
        controllerMap.put("/front-controller/V3/members/new-form", new MemberFormControllerV3());
        controllerMap.put("/front-controller/V3/members/save", new MemberSaveControllerV3());
        controllerMap.put("/front-controller/V3/members", new MemberListControllerV3());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServletV3.service");

        String requestURI = request.getRequestURI();
        ControllerV3 controller = controllerMap.get(requestURI);

        if(controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        // 서블릿 request에서 파라미터 이름을 다 가져온 뒤에 이름마다 데이터를 집어넣는다.
        Map<String, String> paramMap = createParamMap(request);

        // 각 컨트롤러에서 모델을 가져온다.
        ModelView modelView = controller.process(paramMap);
        // 컨트롤러 로직에서 모델에 추가했던 논리 뷰 이름을 가져온다.
        String viewName = modelView.getViewName();

        // 컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변환한다.
        MyView myView = viewResolver(viewName);
        // 화면을 렌더링한다.
        myView.render(modelView.getModel(), request, response);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));

        return paramMap;
    }
}

```

{% endtab %} {% endtabs %}