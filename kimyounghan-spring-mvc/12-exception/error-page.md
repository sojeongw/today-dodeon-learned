# 오류 화면 제공

서블릿 컨테이너가 제공하는 기본 화면은 고객 친화적이지 않다.

{% tabs %} {% tab title="WebServerCustomizer.java" %}

```java

@Component
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {

    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/errorpage/404");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/errorpage/500");

        factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
    }
}
```

{% endtab %} {% endtabs %}

- 404, 500, 혹은 그 외의 에러에 대해 표시할 페이지를 지정할 수 있다.
- 오류 페이지는 예외를 다룰 때 그 자식 타입의 오류를 함꼐 처리한다.
    - ex. RuntimeException + 그의 자식

{% tabs %} {% tab title="ErrorPageController.java" %}

```java

@Slf4j
@Controller
public class ErrorPageController {

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 404");
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 500");
        return "error-page/500";
    }
}
```

{% endtab %} {% endtabs %}

- 매핑된 url에 대해 처리할 컨트롤러를 만든다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%201.33.43.png)

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%201.33.51.png)

1. 스프링 부트가 뜰 때 WebServerCustomizer를 보고 톰캣에 에러 페이지를 등록한다.
2. ServletExController에 매핑된 주소를 호출해 예외가 발생한다.
3. 예외를 서블릿까지 전달한다.
4. WAS가 등록된 에러 페이지를 다시 호출한다.

## 작동 원리

```text
# 예외 발생 시
WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외 발생)

# sendError() 호출 시
WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(response.sendError())
```

- 서블릿은 예외나 sendError()가 호출되었을 때 설정된 오류 페이지를 찾는다.
- WAS는 해당 예외를 처리할 수 있는 오류 페이지 정보를 확인한다.

### 오류 페이지 요청 흐름

```text
WAS(/error-page/500 다시 요청) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error-page/500) -> View
```

- WAS에 예외가 전파되면 오류 페이지를 다시 요청해서 필터, 서블릿, 인터셉터, 컨트롤러 등 모든 과정을 거친다.
- 웹 브라우저(클라이언트)는 서버 내부에서 이런 일이 일어나는지 전혀 모른다.
    - 오직 서버 내부에서 일어나는 일이다.

## 오류 정보 추가

WAS는 단순 오류 페이지 요청 말고도 오류 정보를 request의 attribute에 추가해서 넘겨준다. 필요하다면 오류 페이지에서 사용 가능하다.

{% tabs %} {% tab title="ErrorPageController.java" %}

```java

@Slf4j
@Controller
public class ErrorPageController {

    // RequestDispatcher 상수로 정의된 값들
    public static final String ERROR_EXCEPTION = "javax.servlet.error.exception";
    public static final String ERROR_EXCEPTION_TYPE = "javax.servlet.error.exception_type";
    public static final String ERROR_MESSAGE = "javax.servlet.error.message";
    public static final String ERROR_REQUEST_URI = "javax.servlet.error.request_uri";
    public static final String ERROR_SERVLET_NAME = "javax.servlet.error.servlet_name";
    public static final String ERROR_STATUS_CODE = "javax.servlet.error.status_code";

    private void printErrorInfo(HttpServletRequest request) {
        log.info("ERROR_EXCEPTION: ex=", request.getAttribute(ERROR_EXCEPTION));
        log.info("ERROR_EXCEPTION_TYPE: {}", request.getAttribute(ERROR_EXCEPTION_TYPE));
        log.info("ERROR_MESSAGE: {}", request.getAttribute(ERROR_MESSAGE));
        log.info("ERROR_REQUEST_URI: {}", request.getAttribute(ERROR_REQUEST_URI));
        log.info("ERROR_SERVLET_NAME: {}", request.getAttribute(ERROR_SERVLET_NAME));
        log.info("ERROR_STATUS_CODE: {}", request.getAttribute(ERROR_STATUS_CODE));

        log.info("dispatchType={}", request.getDispatcherType());
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%202.17.16.png)

request.attribute에 서버가 담아준 정보가 잘 출력되었다.