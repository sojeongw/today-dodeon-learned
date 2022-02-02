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