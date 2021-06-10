# V5: 유연한 컨트롤러

```java
public interface ControllerV3 {
    ModelView process(Map<String, String> paramMap);
}
```

```java
public interface ControllerV4 {
    String process(Map<String, String> paramMap, Map<String, Object> model);
}
```

어떤 개발자는 V3로, 어떤 개발자는 V4로 개발하고 싶다면 어떻게 해야할까?

```java

@WebServlet(name = "frontControllerServletV4", urlPatterns = "/front-controller/V4/*")
public class FrontControllerServletV4 extends HttpServlet {

    private Map<String, ControllerV4> controllerMap = new HashMap<>();

    public FrontControllerServletV4() {
        controllerMap.put("/front-controller/V4/members/new-form", new MemberFormControllerV4());
        controllerMap.put("/front-controller/V4/members/save", new MemberSaveControllerV4());
        controllerMap.put("/front-controller/V4/members", new MemberListControllerV4());
        // value의 타입이 맞지 않아 사용할 수 없다!
        controllerMap.put("/front-controller/V3/members", new MemberListControllerV3());
    }
}
```

프론트 컨트롤러에는 이미 버전으로 타입이 박혀있기 때문에 다른 버전을 사용할 수가 없다.

하나의 프로젝트 안에서 V3, V4 등 다양한 버전의 컨트롤러를 돌려 쓰고 싶다면 어댑터 패턴을 이용한다. 110v와 220v 콘센트를 동시에 사용하고 싶다면 어댑터를 사용하듯, 타입이 달라 호환이 불가할 때 사용할
수 있다.

## 어댑터 패턴

![](../../.gitbook/assets/kimyounghan-spring-mvc/04/screenshot%202021-07-19%20오전%207.52.15.png)

ControllerV3, V4처럼 완전히 다른 인터페이스를 호환하기 위해 어댑터 패턴을 적용한다.

### 핸들러 어댑터 목록

핸들러는 컨트롤러라고 생각하면 된다. 핸들러 어댑터 목록은 호환되는 어댑터를 찾아온다. 110v용 어댑터, 220v용 어댑터를 찾아오듯 V3를 처리할 수 있는 어댑터를 찾아오는 것이다.

프론트 컨트롤러가 핸들러 어댑터에서 어댑터를 가지고 왔으면, 예전처럼 컨트롤러를 직접 호출하지 않고 어댑터를 통해서 호출한다. 110v 콘센트를 그대로 쓰지 않고 어댑터를 통해서 사용하듯.

### 핸들러 어댑터

프론트 컨트롤러가 핸들러 어댑터에게 컨트롤러를 넘긴다. 그럼 어댑터는 컨트롤러를 대신 호출해준다. 처리 후 결과인 모델 뷰를 반환한다.

이렇게 핸들러 어댑터 덕분에 다양한 컨트롤러를 호출할 수 있게 된다.

### 핸들러

컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경한 것이다. 꼭 컨트롤러 뿐만 아니라 어떤 종류든 어댑터만 있으면 다 처리할 수 있기 때문이다.

{% tabs %} {% tab title="MyHandlerAdapter.java" %}

```java
public interface MyHandlerAdapter {
    boolean supports(Object handler);

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```

{% endtab %} {% endtabs %}

### supports()

- 핸들러 매핑 정보를 통해 V3인 걸 알았으면 핸들러 어댑터에서 V3용 어댑터를 꺼내와야 한다. 그때 사용하는 것이 supports()다.
    - 즉, 어댑터가 해당 컨트롤러를 처리할 수 있으면 true를 반환한다.

### handle()

- 컨트롤러를 실제로 호출하고 ModelView를 반환한다.
- 컨트롤러가 ModelView를 반환하지 못하면 어댑터가 직접 생성해서라도 반환한다.
- 이전에는 프론트 컨트롤러가 직접 컨트롤러를 호출했지만 이제는 어댑터를 통해 호출한다.

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
public class FrontControllerServletV5 extends HttpServlet {

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

        if (handler == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        // 2. 해당하는 컨트롤러를 가진 어댑터를 가져온다.
        MyHandlerAdapter adapter = getHandlerAdapter(handler);

        // 3. 어댑터를 통해 컨트롤러를 호출하고 model view를 가져온다.
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
            if (adapter.supports(handler)) {
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

## 다른 버전의 핸들러 추가

이제 V3만 있던 프론트 컨트롤러에 V4 컨트롤러를 추가해보자.

{% tabs %} {% tab title="FrontControllerServletV5.java" %}

```java

@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

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

        // V4 핸들러를 추가한다.
        handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new MemberFormControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdaptor());
        // V4의 어댑터를 추가한다.
        handlerAdapters.add(new ControllerV4HandlerAdaptor());
    }
}
```

{% endtab %} {% tab title="ControllerV4HandlerAdaptor.java" %}

```java
public class ControllerV4HandlerAdaptor implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return handler instanceof ControllerV4;
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV4 controller = (ControllerV4) handler;

        Map<String, String> paramMap = createParamMap(request);
        HashMap<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);

        // 콘센트에 어댑터를 끼우듯 ModelView로 타입을 맞추기 위해 어댑터에서 modelView를 만들어준다.
        ModelView modelView = new ModelView(viewName);
        modelView.setModel(model);

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

{% endtab %} {% endtabs %}

- 하나의 프론트 컨트롤러에서 다양한 버전의 컨트롤러를 호출할 수 있게 되었다. 
- 어댑터 덕분에 프론트 컨트롤러의 메인 로직은 건드리지 않고 확장할 수 있었다.
- V4는 뷰 이름을 반환하지만, 어댑터 덕분에 모델 뷰로 변환해주었다.
  - 마치 110V 콘센터를 220v로 변경하듯이!

![](../../.gitbook/assets/kimyounghan-spring-mvc/04/screenshot%202021-07-19%20오전%207.52.15.png)

다시 그림을 보자. 인터페이스를 통하도록 설계했더니 프론트 컨트롤러를 크게 수정하지 않고 OCP를 지키면서 개발할 수 있었다.