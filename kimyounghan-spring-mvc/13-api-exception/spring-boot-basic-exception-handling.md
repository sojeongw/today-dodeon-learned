# 스프링 부트 기본 오류 처리

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%2012.00.35.png)

- 스프링부트는 기본적으로 오류 처리를 해주는 컨트롤러를 제공한다.

### errorHtml()

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%2012.04.03.png)

- Accept 헤더 값이 text/html이면 호출해서 view를 제공한다.

### error()

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%2012.04.30.png)

- 그 외의 경우 호출된다.
- responseEntity로 HTTP Body에 JSON 데이터를 반환한다.

즉, 스프링 부트는 기본적으로 오류가 발생하면 `/error`로 오류 페이지를 요청한다. BasicErrorController는 이 경로를 기본으로 받도록 되어있으며, 기본 경로는
properties에서 `server.error.path`로 수정 가능하다.

```properties
server.error.include-binding-errors=always
server.error.include-exception=true
server.error.include-message=always
server.error.include-stacktrace=always
```

```json
{
  "timestamp": "2021-04-28T00:00:00.000+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "exception": "java.lang.RuntimeException",
  "trace": "java.lang.RuntimeException: 잘못된 사용자 hello.exception.web.api.ApiExceptionControllergetMember(ApiExceptionController.java: 19...",
  "message": "잘못된 사용자",
  "path": "/api/members/ex"
}
```

추가적인 오류 정보를 알기 위해 다양한 옵션을 추가할 수도 있다. 보안상 위험할 수 있으니 간결하게만 노출하고 로그를 통해 확인하자.

## HTML 페이지 vs API 오류

- BasicErrorController를 확장하면 JSON 메시지 변경이 가능하다.
- BasicErrorController는 HTML 페이지를 제공하는 경우는 편리하다.
    - 하지만 API 오류 처리는 예외마다 각각 다른 응답을 던져줘야 할 수 있다.
    - 예를 들어 회원 API와 상품 API의 예외 처리는 각자 다르다.
- 따라서 보통 @ExceptionHandler를 사용하는 게 더 편리하다.
