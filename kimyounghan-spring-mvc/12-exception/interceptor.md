# 인터셉터

{% tabs %} {% tab title="LogInterceptor.java" %}

```java

@Slf4j
public class LogInterceptor implements HandlerInterceptor {

    public static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse
            response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        request.setAttribute(LOG_ID, uuid);

        log.info("REQUEST [{}][{}][{}][{}]", uuid,
                request.getDispatcherType(), requestURI, handler);

        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse
            response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse
            response, Object handler, Exception ex) throws Exception {

        String requestURI = request.getRequestURI();
        String logId = (String) request.getAttribute(LOG_ID);

        log.info("RESPONSE [{}][{}][{}]", logId, request.getDispatcherType(), requestURI);

        if (ex != null) {
            log.error("afterCompletion error!!", ex);
        }
    }
}
```

{% endtab %} {% tab title="WebConfig.java" %}

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // dispatcherType을 지정하는 것 없이 pathPattern으로 조작한다.
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns(
                        "/css/**", "/*.ico"
                        , "/error", "/error-page/**" // 오류 페이지 경로
                );
    }
}
```

{% endtab %} {% endtabs %}

- 인터셉터는 서블릿이 아니라 스프링이 제공하는 기능이기 때문에 DispatcherType과 무관하게 항상 호출된다.
- 대신 요청 경로에서 오류 페이지 경로를 빼주면 된다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%202.53.45.png)

- interceptor는 prehandle() 도중에 에러가 발생하면 postHandle()을 건너뛰고 afterCompletion()으로 가기 때문에 로그가 위처럼 찍혔다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%202.55.24.png)

- exclude 패턴에 넣었으므로 인터셉터가 다시 요청되지 않는다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%202.56.54.png)

- pathPattern에서 제외하면 interceptor가 두 번 호출된다.
- 오류 페이지 호출도 결국 호출이기 때문에 postHandle()로 호출되었다.