# HandlerExceptionResolver

- 발생하는 예외에 따라 상태 코드, 오류 메시지, 형식 등을 다르게 처리할 수 있다.
- HandlerExceptionResolver는 예외가 컨트롤러 밖으로 던져진 경우의 동작을 새로 정의할 수 있다.
- 줄여서 ExceptionResolver라고 한다.

{% tabs %} {% tab title="ApiExceptionController.java" %}

```java

@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }

        return new MemberDto(id, "hello " + id);
    }
}
```

{% endtab %} {% endtabs %}

```json
{
  "status": 500,
  "error": "Internal Server Error",
  "exception": "java.lang.IllegalArgumentException",
  "path": "/api/members/bad"
}
```

- 기존에는 IllegalArgumentException을 400으로 처리하고 싶어도 호출하면 500으로 처리해버린다.

## 적용 전

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%2012.34.53.png)

1. 요청이 컨트롤러에 전달됐다가 실패한다.
2. postHandle()은 건너뛰고 afterCompletion()을 호출한다.
3. WAS에 예외가 전달된다.

## 적용 후

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%2012.35.01.png)

1. 요청이 컨트롤러에 전달됐다가 실패한다.
2. DispatcherServlet에 예외가 반환되면 ExceptionResolver에게 처리할 수 있는지 물어본다.
3. ExceptionResolver가 예외를 해결하려고 시도하고 ModelAndView를 반환한다.
    - 이때 예외를 해결해도 postHandle()은 호출되지 않는다.
4. afterCompletion()을 호출한다.
5. ModelAndView에 따라 뷰를 렌더링 하거나 하지 않고 바로 WAS에 반환한다.
6. WAS는 sendError()가 호출됐음을 알고 오류 페이지를 뒤진다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%2012.40.33.png)

- 핸들러 정보와 발생한 예외를 가지고 처리하는 것을 알 수 있다.

## MyHandlerExceptionResolver

{% tabs %} {% tab title="MyHandlerExceptionResolver.java" %}

```java

@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest request,
                                         HttpServletResponse response,
                                         Object handler,
                                         Exception ex) {
        try {
            // 특정 예외에 대해 정상 처리를 한다.
            if (ex instanceof IllegalArgumentException) {
                log.info("IllegalArgumentException resolver to 400");

                // 예외는 response에 담아 먹어버린다. 즉, 예외를 sendError()로 바꿔치기 한다.
                response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());

                // 정상적으로 ModelAndView를 반환한다.
                return new ModelAndView();
            }
        } catch (IOException e) {
            log.error("resolver ex", e);
        }

        // 예외가 그대로 전달된다.
        return null;
    }
}
```

{% endtab %} {% tab title="WebConfig.java" %}

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {

    ...

    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        // 만든 resolver를 등록해준다.
        resolvers.add(new MyHandlerExceptionResolver());
    }
}
```

{% endtab %}{% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%2012.48.34.png)

- IllegalException이 발생하면 response.sendError(400)을 호출해 상태 코드를 지정하고 빈 ModelAndView를 반환한다.
    - try-catch처럼 예외를 처리해 정상 흐름처럼 변경하기 위해 ModelAndView를 반환한다.

## ExceptionResolver의 반환 값에 따른 DispatcherServlet의 동작 방식

### 빈 ModelAndView

- 뷰를 렌더링 하지 않고 정상 흐름으로 서블릿을 리턴한다.

### ModelAndView 지정

- ModelAndView에 View, Model 정보를 지정해서 반환하면 뷰를 렌더링 한다.

### null

- 다음 ExceptionResolver를 찾아 실행한다.
- 처리할 수 있는 ExceptionResolver가 없으면 예외 처리를 하지 않고 서블릿 밖으로 던진다.

## Exception Resolver 활용

### 예외 상태 코드 변환

- response.sendError() 호출로 상태 코드에 따라 오류를 처리하도록 위임한다.
- WAS는 sendError()를 보고 알맞은 서블릿 오류 페이지를 찾는다.
    - ex. 스프링 부트가 기본으로 설정한 `/error` 페이지를 호출한다.

### 뷰 템플릿 처리

- ModelAndView에 값을 채워서 예외에 따라 새로운 오류 화면을 렌더링 해서 제공한다.

### API 응답 처리

- response.getWriter().println("hello")처럼 HTTP 응답 바디에 직접 데이터를 넣을 수 있다.
    - 여기에 JSON을 넣으면 API 응답 처리를 할 수 있다.

## ExceptionHandler 등록

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
    }
}
```

- 등록 방식에는 2가지가 있다.
    - configureHandlerExceptionResolvers()
        - 스프링이 기본으로 등록하는 ExceptionResolver가 제거된다.
    - extendHandlerExceptionResolvers()
        - 따라서 이 메서드를 활용하자.
