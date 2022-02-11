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