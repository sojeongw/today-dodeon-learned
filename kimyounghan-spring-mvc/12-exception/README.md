# 서블릿 예외 처리

스프링 없는 순수 서블릿 컨테이너의 예외 처리 방식을 알아본다. 서블릿은 2가지에 대해 지원한다.

- Exception
    - 예외
- response.sendError
    - HTTP 상태 코드
    - 오류 메시지

## Exception

### main 메서드를 직접 실행하는 경우

- main이라는 이름의 스레드가 실행된다.
- 실행 도중 예외를 잡지 못하면 상위로 계속 올라간다.
    - main()을 넘어서 예외가 던져지면 예외 정보를 남기고 스레드가 종료된다.

### 웹 애플리케이션을 실행하는 경우

- 사용자 요청 별로 별도의 스레드가 할당되어 서블릿 컨테이너 안에서 실행된다.
- 예외가 발생했을 때
    - try - catch로 잡아서 처리한다.
    - 혹은 잡지 못하고 서블릿 밖으로 전달한다.

```text
WAS(예외 전파됨) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외 발생)
```

- 컨트롤러에서 발생한 예외는 서블릿을 거쳐 WAS까지 전달된다.

{% tabs %} {% tab title="application.properties" %}

```properties
server.error.whitelabel.enabled=false
```

{% endtab %} {% endtabs %}

테스트에 앞서 우선 스프링이 기본적으로 제공하는 white label 페이지를 사용하지 않도록 설정한다.

{% tabs %} {% tab title="ServletExController.java" %}

```java

@Slf4j
@Controller
public class ServletExController {

    @GetMapping("/error-ex")
    public void errorEx() {
        throw new RuntimeException("예외 발생!");
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오전%2011.24.24.png)

- tomcat이 지원하는 기본 오류 화면이 뜬다.
- HTTP 상태 코드는 500을 반환한다.
    - Exception은 서버 내부에서 처리할 수 없는 오류가 발생한 것으로 다룬다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오전%2011.28.35.png)

이번엔 아무 사이트나 호출해본다.

- tomcat이 지원하는 기본 오류 화면이 뜬다.
- HTTP 상태 코드는 404를 반환한다.
    - 해당 url로 반환할 리소스가 없기 때문이다.

## response.sendError

- 오류가 발생하면 HttpServletResponse의 sendError()를 사용하는 방식
    - 서블릿 컨테이너에게 오류 발생을 알릴 수 있다.
- HTTP 상태 코드와 오류 메시지도 추가할 수 있다.

{% tabs %} {% tab title="ServletExController.java" %}

```java

@Slf4j
@Controller
public class ServletExController {

    @GetMapping("/error-404")
    public void error404(HttpServletResponse response) throws IOException {
        response.sendError(404, "404 오류!");
    }

    @GetMapping("/error-500")
    public void error500(HttpServletResponse response) throws IOException {
        response.sendError(500);
    }
}

```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오전%2011.33.17.png)

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오전%2011.33.33.png)

- 예외에는 상태 코드를 넣을 수 없었는데 이제 가능해졌다.

```text
WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(response.sendError())
```

1. sendError를 호출하면 response에 오류가 발생했다는 걸 담아둔다.
2. 서블릿 컨테이너가 응답 전에 sendError()가 호출됐는지 확인한다.
3. 호출됐다면 설정한 오류 코드에 맞춰 페이지를 보여준다.