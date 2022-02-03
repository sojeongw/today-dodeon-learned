# 서블릿 필터

## 공통 관심 사항

로그인을 한 사람만 등록, 수정, 삭제, 조회를 할 수 있어야 한다. 다양한 컨트롤러에서 공통으로 로그인 여부를 확인해야 한다. 이렇게 애플리케이션의 여러 로직에서 공통으로 관심있는 것을 cross-cutting
concern, 공통 관심사라고 한다.

각 컨트롤러에서 로그인 여부를 체크하려고 하면 유지 보수가 어렵다. 스프링 AOP로 해결할 수도 있지만, 웹과 관련된 공통 관심사는 서블릿 필터나 스프링 인터셉터를 사용하는 게 좋다.

웹 관련 공통 관심사는 HTTP 헤더나 URL 정보가 필요하다. 서블릿 필터와 스프링 인터셉터는 HttpServletRequest를 제공하기 때문에 편리하게 이용할 수 있다.

## 서블릿 필터

- 서블릿이 지원하는 문지기 역할
- 여기서 서블릿은 스프링의 경우 디스패처 서블릿으로 생각하면 된다.

### 흐름

```text
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
```

- 필터를 호출한 후 서블릿을 호출한다.
    - 모든 고객의 요청 로그를 남기는 요구사항이 있다면 필터를 사용하면 된다.
- URL 패턴 별로 적용할 수 있다.
    - `/*`는 모든 요청에 필터가 적용된다.

### 제한

- 로그인 사용자
    - HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
- 비 로그인 사용자
    - HTTP 요청 -> WAS -> 필터
    - 적절하지 않은 요청이라 판단하고 서블릿을 호출하지 않는다.

필터가 적절하지 않은 요청이라고 판단하면 거기서 끝을 낼 수 있어 로그인 여부를 체크하기에 딱 좋다.

### 체인

```text
HTTP 요청 -> WAS -> 필터1 -> 필터2 -> 필터3 -> 서블릿 -> 컨트롤러
```

- 여러 필터를 체인으로 구성할 수 있다.
    - ex. 로그 필터 -> 로그인 여부 체크 필터

```java
public interface Filter {
    public default void init(FilterConfig filterConfig) throws ServletException {
    }

    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException;

    public default void destroy() {
    }
}
```

- init()
    - 필터 초기화 메서드
    - 서블릿 컨테이너가 생성될 때 호출된다.
- doFilter()
    - 필터의 로직을 구현한다.
    - 고객의 요청이 올 때 마다 호출된다.
        - WAS에서 doFilter()를 먼저 요청한 뒤에 여러 필터를 통과하고 서블릿을 호출한다.
- destroy()
    - 필터 종료 메서드
    - 서블릿 컨테이너가 종료될 때 호출된다.

필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고 관리한다.

## Log 필터 구현

{% tabs %} {% tab title="LogFilter.java" %}

```java

@Slf4j
public class LogFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("log filter doFilter");

        // 자식인 HttpServletRequest로 다운캐스팅 해야 한다.
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        // 각 HTTP 요청을 구분하기 위해 요청당 UUID를 할당한다.
        String uuid = UUID.randomUUID().toString();

        try {
            log.info("REQUEST [{}][{}]", uuid, requestURI);
            // 다음 필터를 꼭 호출해줘야 한다.
            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        } finally {
            // 필터를 호출하고 나면 실행된다.
            log.info("RESPONSE [{}][{}]", uuid, requestURI);
        }
    }

    @Override
    public void destroy() {
        log.info("log filter destroy");
    }
}
```

{% endtab %} {% endtabs %}

- 필터를 사용하려면 Filter 인터페이스를 구현해야 한다.
- HTTP 요청이 오면 doFilter()가 호출된다.
- chain.doFilter();
    - 다음 필터가 있으면 필터를 호출하고 필터가 없으면 서블릿을 호출한다.
    - 이 로직이 없으면 다음 단계로 진행되지 않는다.

{% tabs %} {% tab title="WebConfig.java" %}

```java

@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();

        // 만들어 둔 필터를 추가한다.
        filterRegistrationBean.setFilter(new LogFilter());
        // 필터는 체인으로 동작하기 때문에 순서를 지정해줘야 한다.
        filterRegistrationBean.setOrder(1);
        // 모든 url 패턴에 적용한다.
        filterRegistrationBean.addUrlPatterns("/*");

        return filterRegistrationBean;
    }
}
```

{% endtab %} {% endtabs %}

- 필터를 실제로 사용하려면 등록이 필요하다.
    - 스프링 부트를 쓴다면 FilterRegistrationBean으로 등록한다.

```java
@ServletComponentScan @WebFilter(filterName = "logFilter", urlPatterns = "/*")
```

필터를 이렇게도 등록할 수 있지만 필터 순서 조절이 안되므로 FilterRegistrationBean을 쓰자.

```text
log filter doFilter
REQUEST [e6ad5d37-f4c6-4047-a899-610aa2416783][/]
RESPONSE [e6ad5d37-f4c6-4047-a899-610aa2416783][/]

log filter doFilter
REQUEST [5eaf3f1b-7d5d-429f-baae-28b56956c4cd][/login]
RESPONSE [5eaf3f1b-7d5d-429f-baae-28b56956c4cd][/login]
```

실무에서 같은 HTTP 요청에 대해 같은 식별자를 남기고 싶다면 logback mdc로 검색해보자.