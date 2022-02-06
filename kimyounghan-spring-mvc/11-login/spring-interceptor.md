# 스프링 인터셉터

## 흐름

서블릿 필터가 서블릿이 제공하는 기술이라면 스프링 인터셉터는 스프링 MVC가 제공한다. 둘 다 웹의 공통 관심 사항얼 처리하지만 순서, 범위, 방법이 다르다.

```text
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
```

- 스프링 인터셉터는 서블릿과 컨트롤러 사이에서 호출된다.
- 스프링 인터셉터가 스프링 MVC의 기능이므로 여기서의 서블릿은 디스패처 서블릿을 의미한다.
    - 스프링 MVC의 시작점이 디스패처 서블릿이라고 생각해보면 결국 스프링 인터셉터는 디스패처 서블릿 이후에 등장하는 게 맞다.
- URL 패턴을 적용할 수 있다.
    - 서블릿 URL 패턴과는 다르다.
    - 아주 정밀하게 설정할 수 있다.

## 제한

- 로그인 사용자
    - HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
- 비로그인 사용자
    - HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터
    - 적절하지 않은 요청이라 판단하고 컨트롤러를 호출하지 않는다.

적절하지 않은 요청은 컨트롤러 직전에 끝낼 수 있으므로 로그인 여부를 체크하기 좋다.

## 체인

```text
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> 컨트롤러
```

- 체인으로 구성된다.
- 중간에 인터셉터를 자유롭게 추가할 수 있다.

## HandlerInterceptor

```java
public interface HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request,
                              HttpServletResponse response,
                              Object handler) throws Exception {
    }

    default void postHandle(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler,
                            @Nullable ModelAndView modelAndView) throws Exception {
    }

    default void afterCompletion(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Object handler,
                                 @Nullable Exception ex) throws Exception {
    }
}
```

서블릿 필터는 doFilter()만 단순하게 호출하고 실수로 호출을 안해서 에러가 나는 경우도 있다. 스프링 인터셉터는 좀 더 세분화 되어 있다.

- preHandle()
    - 컨트롤러 호출 전
- postHandle()
    - 컨트롤러 호출 후
- afterCompletion()
    - 요청 완료 이후

또, 서블릿 필터는 request, response만 제공했지만 스프링 인터셉터는 다양한 정보를 받을 수 있다.

- handler
    - 호출 정보
    - 어떤 컨트롤러가 호출되는지
- modelAndView
    - 응답 정보
    - 어떤 modelAndView가 반환되는지

## 호출 흐름

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오후%204.43.51.png)

1. preHandle
    - 핸들러 어댑터 및 컨트롤러 호출 전에 호출한다.
    - preHandle의 응답이 true면 다음으로 진행한다.
    - false면 나머지 인터셉터와 핸들러 어댑터 모두 호출되지 않고 여기서 끝난다.
2. handle(handler)
3. ModelAndView 반환
4. postHandle
    - 핸들러 어댑터 및 컨트롤러 호출 후에 호출한다.
5. render(model)
6. afterCompletion
    - 뷰가 렌더링 된 이후 호출한다.

## 예외 상황

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오후%204.43.59.png)

1. preHandle
    - 컨트롤러 호출 전에 호출되므로 예외 발생 여부와 상관 없이 실행된다.
2. handle(handler)
3. ModelAndView 반환
4. postHandle
    - 컨트롤러에서 예외가 발생하면 호출되지 않는다.
5. render(model)
6. afterCompletion
    - 예외와 무관하게 항상 호출된다.
    - 예외를 파라미터로 받아서 어떤 예외가 발생했는지 로그로 출력할 수 있다.
    - 예외와 무관하게 공통 처리를 하려면 여기를 이용해야 한다.

스프링 인터셉터는 스프링 MVC 구조에 특화된 필터 기능이다. 스프링 MVC를 사용하고 특별히 필터를 써야 하는 상황이 아니라면 인터셉터를 사용하는 게 더 편하다.

## 요청 로그 인터셉터 구현하기

{% tabs %} {% tab title="LogInterceptor.java" %}

```java

@Slf4j
public class LogInterceptor implements HandlerInterceptor {

    private static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             // 어떤 컨트롤러가 호출되는지도 확인할 수 있다.
                             Object handler) throws Exception {

        // 처음부터 HttpServletRequest로 들어오므로 캐스팅이 필요없어 편하다.
        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        /*
        afterCompletion()로 uuid 값을 전달하고 싶은데 방법이 없다.
        지역 변수로 뽑으면 싱글톤이기 때문에 위험하다.
        setAttribute()를 쓰면 해결된다.
        */
        request.setAttribute(LOG_ID, uuid);

        /*
        @RequestMapping을 사용하면 HandlerMethod에 핸들러 정보가 담겨온다.
        정적 리소스를 사용하면 ResourceHttpRequestHandler가 사용된다.
        */
        if (handler instanceof HandlerMethod) {
            // 호출할 컨트롤러 메서드의 모든 정보가 포함되어 있다.
            HandlerMethod hm = (HandlerMethod) handler;
        }

        log.info("REQUEST [{}][{}][{}]", uuid, requestURI, handler);

        // false면 여기서 끝나고 true면 다음 컨트롤러가 호출된다.
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) throws Exception {

        // HTTP Request는 갔다가 돌아올 때까지 같은 요청인 게 보장되므로 uuid도 그대로 받을 수 있다.
        String logId = (String) request.getAttribute(LOG_ID);
        String requestURI = request.getRequestURI();

        log.info("RESPONSE [{}][{}]", logId, requestURI);

        if (ex != null) {
            log.error("afterCompletion error!!", ex);
        }
    }
}
```

{% endtab %} {% endtabs %}

- uuid
    - 요청 로그를 구분하기 위해 생성한다.
- request.setAttribute(LOG_ID, uuid)
    - 서블릿 필터는 지역 변수로 해결할 수 있지만 스프링 인터셉터는 호출 시점이 완전 분리되어 있어 불가하다.
    - 다른 메서드에서도 함께 사용하려면 request에 담아두면 된다.
        - LogInterceptor는 싱글톤처럼 사용되기 때문에 멤버 변수로 사용하면 위험하다.
- return true
    - 정상 호출
    - 다음 인터셉터나 컨트롤러가 호출된다.
- afterCompletion()
    - 예외 발생 시 postHandle()은 호출되지 않기 때문에 여기에 종료 로그를 남긴다.

### handlerMethod

- 어떤 핸들러 매핑을 사용하는가에 따라 핸들러 정보가 달라진다.
    - 일반적으로 @Controller, @RequestMapping으로 핸들러 매핑을 사용한다.
    - 이 경우 핸들러 정보로 HandlerMethod가 넘어온다.
- ResourceHttpRequestHandler
    - @Controller 대신 `/resources/static` 같은 정적 리소스는 여기로 핸들러 정보가 넘어온다.

### 인터셉터 등록

{% tabs %} {% tab title="WebConfig.java" %}

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/*.ico", "/error");
    }
}
```

{% endtab %} {% endtabs %}

- WebMvcConfigurer의 addInterceptors()으로 인터셉터를 등록한다.
- addPathPatterns()
    - 인터셉터를 적용할 URL 패턴 지정
- excludePathPatterns()
    - 인터셉터에서 제외할 패턴 지정

[PathPattern 공식 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/pattern/PathPattern.html)

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오후%205.38.41.png)

- LoginCheckFilter 뒤에 LogInterceptor가 실행되었다.
    - ModelAndView부터 핸들러 정보, Member, Model 등의 파라미터 정보까지 다 찍힌다.