# 스프링 부트 오류 페이지

스프링 부트는 복잡한 에러 페이지 설정 과정을 기본으로 제공한다.

- ErrorPage를 자동으로 등록한다.
    - ErrorMvcAutoConfiguration이 오류 페이지를 자동 등록한다.
    - /error 라는 경로로 기본 오류 페이지를 설정한다.
        - 별도의 ErrorPage나 상태코드, 예외를 설정하지 않으면 기본을 사용한다.
        - 서블릿 밖으로 예외가 발생하거나 sendError()가 호출되면 모든 오류는 /error를 호출한다.
- 스프링 컨트롤러 BasicErrorController를 호출한다.
    - ErrorPage에서 등록한 /error를 매핑해서 처리한다.
    - 기본적인 로직이 모두 있기 때문에 개발자는 룰과 우선순위에 따라 등록만 하면 된다.

## 뷰 선택 우선 순위

1. 뷰 템플릿
    1. resources/templates/error/500.html
    2. resources/templates/error/5xx.html
2. 정적 리소스
    - static, public 폴더
        1. resources/static/error/400.html
        2. resources/static/error/404.html
        3. resources/static/error/4xx.html
3. 적용 대상이 없을 때
    1. resources/templates/error.html

해당 경로에 HTTP 상태 코드 이름으로 뷰 파일을 넣어두면 된다. 404 등 구체적인 것이 4xx 등 포괄적인 것보다 우선 순위가 높다.

## BasicErrorController의 기본 정보

```text
* timestamp: Fri Feb 05 00:00:00 KST 2021
* status: 400
* error: Bad Request
* exception: org.springframework.validation.BindException
* trace: 예외 trace
* message: Validation failed for object='data'. Error count: 1
* errors: Errors(BindingResult)
* path: 클라이언트 요청 경로 (`/hello`)
```

![](../../.gitbook/assets/kimyounghan-spring-mvc/12/screenshot%202022-03-19%20오후%203.49.26.png)

- BasicErrorController는 위의 정보를 model에 담아 뷰에 전달한다.
- 뷰 템플릿에서 이 값을 활용해 출력할 수 있다.
- 하지만 고객은 이해할 수 없는 내용이고 보안상 문제가 될 수도 있어 웬만하면 막아 놓는다.

{% tabs %} {% tab title="application.properties" %}

```properties
# exception 포함 여부
server.error.include-exception=false
# message 포함 여부
server.error.include-message=never
# trace 포함 여부
server.error.include-stacktrace=never
# errors 포함 여부
server.error.include-binding-errors=never
```

{% endtab %} {% endtabs %}

오류 정보의 포함 여부는 properties에서 설정할 수 있다.

```properties
server.error.include-exception=true
server.error.include-message=on_param
server.error.include-stacktrace=on_param
server.error.include-binding-errors=on_param
```

값이 never인 부분은 3가지 옵션이 가능하다.

- never
    - 사용하지 않음
- always
    - 항상 사용함
- on_param
    - 파라미터가 있을 때 사용함
    - 디버깅할 때 사용할 수 있어 개발 서버엔 써도 되지만 운영 서버는 권장하지 않는다.

### on_param

```text
http://localhost:8080/error-ex?message=&errors=&trace=
```

- HTTP 요청할 때 파라미터를 전달하면 정보들이 model에 담겨 뷰 템플릿으로 출력된다.
- 실무에서는 절대 노출하면 안된다.
- 고객이 이해할 수 있는 친절한 메시지로 남겨야 한다.
    - 오류는 서버에 로그로 남겨 확인한다.

## 스프링 부트 오류 관련 옵션

- server.error.whitelabel.enabled=true
    - 오류 처리 화면을 못 찾으면 스프링 whitelabel 페이지를 적용한다.
- server.error.path=/error
    - 오류 페이지 경로 지정
    - 스프링이 자동으로 등록하는 서블릿 글로벌 오류 페이지 경로와 BasicErrorController의 경로로 함께 사용된다.

## 확장 포인트

에러를 공통으로 처리하는 컨트롤러의 기능을 변경하고 싶다면 ErrorController 인터페이스나 BasicErrorController를 상속 받는다.