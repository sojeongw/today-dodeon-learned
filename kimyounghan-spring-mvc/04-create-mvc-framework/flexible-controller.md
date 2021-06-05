# V5: 유연한 컨트롤러

하나의 프로젝트 안에서 V3, V4 등 다양한 버전의 컨트롤러를 돌려 쓰고 싶다면 어댑터 패턴을 이용한다.

## 어댑터 패턴

![](../../.gitbook/assets/kimyounghan-spring-mvc/04/screenshot%202021-07-19%20오전%207.52.15.png)

ControllerV3, V4처럼 완전히 다른 인터페이스를 호환하기 위해 어댑터 패턴을 적용한다.

### 핸들러 어댑터

어댑터를 통해 다양한 종류의 컨트롤러를 호출한다.

### 핸들러

컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경한다. 어댑터가 생겼기 때문에 꼭 컨트롤러 뿐만 아니라 해당 종류의 어댑터만 있으면 뭐든 다 처리할 수 있기 때문이다.

{% tabs %} {% tab title="MyHandlerAdapter.java" %}

```java
public interface MyHandlerAdapter {
    boolean supports(Object handler);
    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```

{% endtab %} {% endtabs %}

### supports

- handler는 컨트롤러를 의미한다.
- 어댑터가 해당 컨트롤러를 처리할 수 있으면 true를 반환한다.

### handle

- 실제 컨트롤러를 호출하고 ModelView를 반환한다.
- 컨트롤러가 ModelView를 반환하지 못하면 어댑터가 직접 생성해서라도 반환한다.
- 이전에는 프론트 컨트롤러가 컨트롤러를 호출했지만 이제는 어댑터를 통해 호출한다.

{% tabs %} {% tab title="ControllerV3HandlerAdaptor.java" %}

```java
public class ControllerV3HandlerAdaptor implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        // V3인지 확인한다.
        return handler instanceof ControllerV3;
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        // support()로 이미 V3인 것을 확인했기 때문에 캐스팅할 수 있다.
        ControllerV3 controller = (ControllerV3) handler;

        Map<String, String> paramMap = createParamMap(request);
        ModelView modelView = controller.process(paramMap);

        return modelView;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));

        return paramMap;
    }
}
```

{% endtab %} {% tab title="FrontControllerServletV5.java" %}

```java
@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5  extends HttpServlet {

    private final Map<String, Object> handlerMappingMap = new HashMap<>();
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();

    public FrontControllerServletV5() {
        initHandlerMappingMap();
        initHandlerAdapters();

    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdaptor());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 핸들러를 찾아온다.
        Object handler = getHandler(request);

        if(handler == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        // 2. 해당하는 컨트롤러를 가진 어댑터를 가져온다.
        MyHandlerAdapter adapter = getHandlerAdapter(handler);

        // 3. 어댑터를 통해 컨트롤러에서 model view를 가져온다.
        ModelView modelView = adapter.handle(request, response, handler);

        String viewName = modelView.getViewName();
        MyView myView = viewResolver(viewName);
        myView.render(modelView.getModel(), request, response);
    }

    private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        Object handler = handlerMappingMap.get(requestURI);

        return handler;
    }

    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if(adapter.supports(handler)) {
                return adapter;
            }
        }
        throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다.");
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```

{% endtab %} {% endtabs %}


