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