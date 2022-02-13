# 필터

```text
1. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외 발생)
2. WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error-page/500) -> View
```

- 오류가 발생하면 WAS 내부에서 다시 호출이 발생하기 때문에 필터와 인터셉트도 한 번 더 호출된다.
- 이런 비효율적인 과정을 방지하기 위해 서블릿은 DispatcherType을 제공한다.

## javax.servlet.DispatcherType

```java
public enum DispatcherType {
    FORWARD,
    INCLUDE,
    REQUEST,
    ASYNC,
    ERROR
}
```

- REQUEST
    - 클라이언트 요청
- ERROR
    - 오류 요청
- FORWARD
    - 서블릿에서 다른 서블릿이나 JSP를 호출할 때
    - RequestDispatcher.forward(request, response);
- INCLUDE
    - 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때
    - RequestDispatcher.include(request, response);
- ASYNC
    - 서블릿 비동기 호출

{% tabs %} {% tab title="LogFilter.java" %}

```java

@Slf4j
public class LogFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        try {
            // dispatcherType을 가져온다.
            log.info("REQUEST [{}][{}][{}]", uuid, request.getDispatcherType(), requestURI);
            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        } finally {
            log.info("RESPONSE [{}][{}][{}]", uuid, request.getDispatcherType(), requestURI);
        }
    }

    @Override
    public void destroy() {
        log.info("log filter destroy");
    }
}
```

{% endtab %} {% tab title="WebConfig.java" %}

```java

@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();

        // 만들어둔 LogFilter를 필터로 등록한다.
        filterRegistrationBean.setFilter(new LogFilter());
        filterRegistrationBean.setOrder(1);
        filterRegistrationBean.addUrlPatterns("/*");
        // 필터가 작동할 타입을 명시한다. 넣지 않으면 기본값은 REQUEST로 들어간다.
        filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);

        return filterRegistrationBean;
    }
}
```

{% endtab %} {% endtabs %}

- 이제 요청과 에러에만 필터가 호출된다.
- 아무것도 넣지 않으면 REQUEST가 기본 값이 된다.
    - 즉, 클라이언트 요청이 있는 경우만 적용한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%202.38.33.png)

- 최초 호출 시 LogFilter가 요청되었다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%202.38.44.png)

- 오류 페이지를 요청하면서 LogFilter를 다시 요청한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%202.38.53.png)

- 오류 페이지를 반환하는 컨트롤러를 호출할 때 request를 까보면 DispatcherType에 ERROR가 담겨있다.

- ![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%202.45.04.png)

- DispatcherType 설정을 빼면 LogFilter가 한 번만 호출된다.
