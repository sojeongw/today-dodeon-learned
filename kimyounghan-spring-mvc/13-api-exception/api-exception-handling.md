# API 예외 처리

- API는 단순 오류 페이지 반환보다 고려할 사항이 많다.
    - 모바일인지 웹인지
    - MSA끼리의 통신인지
    - 다른 회사에 기능을 제공하는 API인지
- 오류 페이지는 화면만 보여주면 끝이지만 API는 각 오류 상황에 맞게 스펙을 정해야 한다.

{% tabs %} {% tab title="ApiExceptionController.java" %}

```java

@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }

        return new MemberDto(id, "hello " + id);
    }

    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String memberId;
        private String name;
    }
}
```

{% endtab %} {% tab title="WebServerCustomizer.java" %}

```java

@Component
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {

    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");

        factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
    }
}
```

{% endtab %} {% endtabs %}

- WebServerCustomizer
    - WAS에 예외가 전달되거나 response.sendError()가 호출되면 등록한 예외 페이지가 호출된다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-26%20오전%2011.36.21.png)

- 예외가 발생하면 JSON으로 반환해야 한다.
- 하지만 이 컨트롤러는 우리가 미리 만들었던 HTML 오류 페이지를 반환한다.
- 웹 브라우저가 아닌 클라이언트라면 이걸 받아서 할 수 있는 게 없다.

{% tabs %} {% tab title="before" %}

```java

@Slf4j
@Controller
public class ErrorPageController {

    ...

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 404");
        printErrorInfo(request);
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 500");
        printErrorInfo(request);
        return "error-page/500";
    }
}
```

{% endtab %} {% tab title="after" %}

```java

@Slf4j
@Controller
public class ErrorPageController {

    ...

    /*
    produces에 springframework의 MediaType을 정하면
    accept가 application/json인 요청이 들어올 경우
    이 메서드를 우선으로 호출한다.
    */
    @RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Map<String, Object>> errorPage500Api(
            HttpServletRequest request, HttpServletResponse response) {
        log.info("API errorPage 500");

        Map<String, Object> result = new HashMap<>();
        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);

        result.put("status", request.getAttribute(ERROR_STATUS_CODE));
        result.put("message", ex.getMessage());

        Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);

        // JSON으로 반환하니까 ResponseEntity를 사용한다.
        return new ResponseEntity(result, HttpStatus.valueOf(statusCode));
    }
}
}
```

{% endtab %}{% endtabs %}

- HTML 화면을 날리는 이유는 ErrorPageController에 정의된 메서드가 호출되기 때문이다.
- produces를 정의해 클라이언트가 받고 싶어하는 미디어 타입에 따라 메서드를 호출하게 변경한다.
- JSON으로 반환하기 위해 ResponseEntity를 사용한다.
    - 메시지 컨버터가 동작하면서 데이터를 JSON으로 변환한다.

### produces = MediaType.APPLICATION_JSON_VALUE

- 클라이언트가 요청하는 HTTP header의 Aceept값이 application/json인 경우 해당 메서드를 호출한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-26%20오전%2011.51.22.png)

- Accept 헤더를 지정한 뒤 요청하면 JSON으로 응답을 받게 된다.