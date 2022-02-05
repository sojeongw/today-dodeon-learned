# 핸들러 매핑과 핸들러 어댑터

{% tabs %} {% tab title="Controller.java" %}

```java
public interface Controller {
    ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

{% endtab %} {% endtabs %}

예전에 스프링이 제공하던 Controller 인터페이스다. @Controller와는 완전히 다르다.

## Controller

{% tabs %} {% tab title="OldController.java" %}

```java
// org.springframework.stereotype.Controller는 애너테이션이므로
// 이걸로 import 해야 한다.

import org.springframework.web.servlet.mvc.Controller;

// OldController 스프링 빈의 이름이 /springmvc/old-controller가 된다.
@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("OldController.handleRequest");
        return null;
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/05/screenshot%202022-02-13%20오전%2011.40.25.png)

- http://localhost:8080/springmvc/old-controller로 접속하면 이 url이 매핑된 핸들러를 찾아야 한다.
    - 핸들러를 찾으려면 핸들러 매핑이 필요하다.
    - 즉, 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
    - url에 스프링 빈의 이름을 넣었으므로 이 이름으로 스프링 빈을 찾아 핸들러를 꺼내오는 것이다.
- 핸들러 매핑으로 찾은 핸들러를 실행할 수 있는 핸들러 어댑터도 필요하다.
    - 인터페이스인 Controller를 호출할 수 있는 어댑터를 찾고 실행해야 한다.

스프링은 이미 핸들러 매핑과 핸들러 어댑터를 다 구현해놨기 때문에 우리가 직접 개발할 일은 거의 없다.

### 스프링 부트

스프링 부트는 핸들러 매핑과 핸들러 어댑터를 자동으로 등록해준다.

- HandlerMapping
    - 0순위: RequestMappingHandlerMapping
        - 애너테이션 기반 컨트롤러인 @RequestMapping에서 사용한다.
    - 1순위: BeanNameUrlHandlerMapping
        - 스프링 빈 이름으로 핸들러를 찾는다.
        - 우리가 바로 직전에 했던 방법이다.
- HandlerAdapter
    - 0순위: RequestMappingHandlerAdapter
        - 애너테이션 기반 컨트롤러인 @RequestMapping에서 사용한다.
    - 1순위: HttpRequestHandlerAdapter
        - HttpRequestHandler를 처리한다.
    - 2순위: SimpleControllerHandlerAdapter
        - Controller 인터페이스를 처리한다.
        - 우리가 바로 직전에 했던 방법이다.

### 실행 과정

![](../../.gitbook/assets/kimyounghan-spring-mvc/05/screenshot%202022-02-14%20오후%2010.39.42.png)

1. 핸들러 매핑으로 핸들러 조회
    1. HandlerMapping을 순서대로 실행해서 핸들러를 찾는다.
    2. 우리의 경우 빈 이름을 핸들러를 찾아야 하므로 BeanNameUrlHandlerMapping을 실행한다.
    3. 핸들러인 OldController 반환한다.
2. 핸들러 어댑터 조회
    1. HandlerAdapter의 supports()를 순서대로 호출한다.
    2. Controller 인터페이스를 지원하는 SimpleControllerHandlerAdapter가 대상이 된다.
3. 핸들러 어댑터 실행
    1. DispatcherServlet이 조회한 SimpleControllerHandlerAdapter를 실행하면서 핸들러 정보를 함께 넘긴다.
    2. SimpleControllerHandlerAdapter는 핸들러인 OldController를 내부에서 실행하고 결과를 반환한다.

### 정리

- HandlerMapping
    - BeanNameUrlHandlerMapping
- HandlerAdapter
    - SimpleControllerHandlerAdapter

## HttpRequestHandler

![](../../.gitbook/assets/kimyounghan-spring-mvc/05/screenshot%202022-02-14%20오후%2010.47.56.png)

이번에는 HttpRequestHandler를 사용해보자.

{% tabs %} {% tab title="MyHttpRequestHandler.java" %}

```java

@Component("/springmvc/request-handler")
public class MyHttpRequestHandler implements HttpRequestHandler {
    @Override
    public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("MyHttpRequestHandler.handleRequest");
    }
}

```

{% endtab %} {% endtabs %}

### 실행 과정

1. 핸들러 매핑으로 핸들러 조회
    1. HanlderMapping을 순서대로 실행해서 빈 이름인 `/springmvc/request-handler`로 핸들러를 찾는다.
    2. 빈 이름으로 핸들러를 찾아야 하므로 BeanNameUrlHandlerMapping을 실행한다.
    3. 핸들러인 MyHttpRequestHandler를 반환한다.
2. 핸들러 어댑터 조회
    1. HandlerAdapter의 supports()를 순서대로 호출한다.
    2. HttpRequestHandler 인터페이스를 지원하는 HttpRequestHandlerAdapter가 대상이 된다.
3. 핸들러 어댑터 실행
    1. DispatcherServlet이 조회한 HttpRequestHandlerAdapter를 실행하면서 핸들러 정보를 함께 넘긴다.
        - DispatcherServlet.doDispatch() 에서 HttpRequestHandlerAdapter.handle()이 호출된다.
    2. HttpRequestHandlerAdapter는 핸들러인 MyHttpRequestHandler를 내부에서 실행하고 결과를 반환한다.

### 정리

- HandlerMapping
    - BeanNameUrlHandlerMapping
- HandlerAdapter
    - HttpRequestHandlerAdapter

## @RequestMapping

- 가장 우선 순위가 높은 핸들러 매핑은 RequestMppingHandlerMapping, 핸들러 어댑터는 RequestMappingHandlerAdapter
- @RequestMapping에서 따온 이름이다.
- 실무에서는 99.9% 이 방식을 사용한다.