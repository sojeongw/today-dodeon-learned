# ExceptionResolver

- 우리가 직접 ExceptionResolver를 구현하는 건 복잡하다.
- 스프링이 제공하는 ExceptionResolver를 사용해보자.

스프링은 다음의 우선 순위에 따라 HandlerExceptionResolverComposite에 ExceptionResolver를 등록한다.

1. ExceptionHandlerExceptionResolver
    - @ExceptionHandler를 처리한다.
    - API 예외 처리를 대부분 해결한다.
2. ResponseStatusExceptionResolver
    - HTTP 상태 코드를 지정한다.
    - ex. `@ResponseStatus(value = HttpStatus.NOT_FOUND)`
3. DefaultHandlerExceptionResolver
    - 스프링 내부에서 발생하는 예외를 처리한다.

## ResponseStatusExceptionResolver

- 예외에 따라 HTTP 상태 코드를 지정해준다.
    - @ResponseStatus에 달려있는 예외
    - ResponseStatusException 예외

```java

@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류")
public class BadRequestException extends RuntimeException {
}
```

```json
{
  "status": 400,
  "error": "Bad Request",
  "exception": "hello.exception.exception.BadRequestException",
  "message": "잘못된 요청 오류",
  "path": "/api/response-status-ex1"
}
```

- 원래 500 에러로 떨어지는 것을 400으로 떨어지도록 처리할 수 있다.

```java

@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "error.bad")
public class BadRequestException extends RuntimeException {
}
```

```properties
# messages.properties에 정의한다.
error.bad=잘못된 요청 오류입니다. 메시지 사용
```

```json
{
  "status": 400,
  "error": "Bad Request",
  "exception": "hello.exception.exception.BadRequestException",
  "message": "잘못된 요청 오류입니다. 메시지 사용",
  "path": "/api/response-status-ex1"
}
```

- reason을 MessageSource에서 찾아올 수도 있다.
    - messages.properties에 메시지를 정의하고 reason에 넣어준다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%202.46.00.png)

- ResponseStatusExceptionResolver 코드를 뜯어보면 sendError()를 호출한다.
    - 따라서 WAS에서 다시 오류 페이지 /error를 요청한다.
- messageSource에서 reason을 찾아오는 것도 볼 수 있다.

### ResponseStatusException

- @ResponseStatus는 개발자가 수정할 수 있는 예외에만 적용할 수 있다.
    - 라이브러리 코드에는 적용할 수 없다.
- 애너테이션이기 때문에 조건에 따라 동적으로 변경하는 것도 어렵다.

```java

@Slf4j
@RestController
public class ApiExceptionController {
    
    ...

    @GetMapping("/api/response-status-ex2")
    public String responseStatusEx2() {
        throw new ResponseStatusException(
                HttpStatus.NOT_FOUND, "error.bad",
                new IllegalArgumentException()
        );
    }
}
```

```json
{
  "status": 404,
  "error": "Not Found",
  "exception": "org.springframework.web.server.ResponseStatusException",
  "message": "잘못된 요청 오류입니다. 메시지 사용",
  "path": "/api/response-status-ex2"
}
```

- ResponseStatusException을 사용하면 상태 코드와 메시지를 똑같이 처리할 수 있다.

## DefaultHandlerExceptionResolver

- 스프링 내부에서 발생하는 스프링 예외를 해결한다.
    - 파라미터 바인딩 시점에 타입이 맞지 않으면 TypeMismatchException이 발생한다.
    - 일반적으로 예외는 500을 던지지만 파라미터 바인딩은 클라이언트에서 잘못 보낸 것이므로 400을 보내주는 게 좋다.
    - DefaultHandlerExceptionResolver는 이럴 때 500에서 400으로 변경해 응답한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%203.13.20.png)

- 코드를 까보면 결국 response.sendError()로 해결한다.
- 따라서 WAS에서 다시 오류 페이지를 요청할 것이다.

```java

@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/default-handler-ex")
    public String defaultException(@RequestParam Integer data) {
        return "ok";
    }
}
```

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%203.07.47.png)

- 일부러 틀린 타입으로 요청하면 400 에러로 찍히는 걸 확인할 수 있다.

## ExceptionHandlerExceptionResolver

### API 예외 처리의 어려운 점

- HandlerExceptionResolver가 반환하는 ModelAndView는 API 응답에 필요하지 않다.
- API 응답을 위해 HttpServletResponse에 응답을 직접 넣어주는 것은 불편하다.
- 특정 컨트롤러에서만 발생하는 예외를 별도로 처리하기가 어렵다.
    - 회원 컨트롤러와 상품 컨트롤러가 동일한 예외를 던져도 다르게 처리해야 한다.

이 문제를 해결하기 위해 스프링은 ExceptionHandlerExceptionResolver를 사용하는 @ExceptionHandler를 제공한다.

{% tabs %} {% tab title="ApiExceptionV2Controller.java" %}

```java

@Slf4j
@RestController
public class ApiExceptionV2Controller {

    // 이 컨트롤러 안에서 IllegalArgumentException이 터지면 이 메서드가 호출된다.
    @ExceptionHandler(IllegalArgumentException.class)
    // RestController이기 때문에 ErrorResult가 그대로 JSON으로 반환된다.
    public ErrorResult illegalExHandle(IllegalArgumentException e) {
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }

    @GetMapping("/api2/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }

        if (id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }

        if (id.equals("user-ex")) {
            throw new UserException("사용자 오류");
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

{% endtab %} {% tab title="ErrorResult.java" %}

```java

@Data
@AllArgsConstructor
public class ErrorResult {
    private String code;
    private String message;
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/13/screenshot%202022-03-26%20오후%203.39.16.png)

- ExceptionHandler에 정의된 대로 메시지가 출력되었다.
- 참고로 해당 예외를 잡는 것은 정의된 컨트롤러 내부만 적용된다.

### @ExceptionHandler 처리 과정

1. 예외가 발생한다.
2. DispatcherServlet으로 반환되면 ExceptionResolver에서 해결할 수 있는지 물어본다.
3. 우선 순위가 제일 높은 ExceptionHandlerExceptionResolver를 실행한다.
4. ExceptionHandlerExceptionResolver가 컨트롤러에 @ExceptionHandler가 붙은 메서드가 있는지 찾는다.
5. 해당 메서드를 호출한다.

서블릿 컨테이너로 다시 거슬러 올라가지 않고 정상 응답 후 여기서 흐름이 다 끝난다. 하지만 정상 흐름으로 바꾸는 과정이기 때문에 응답은 200으로 나간다.

{% tabs %} {% tab title="ApiExceptionV2Controller.java" %}

```java

@Slf4j
@RestController
public class ApiExceptionV2Controller {

    // 200 대신 다른 HTTP 상태 코드를 반환하고 싶을 때 사용한다.
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandle(IllegalArgumentException e) {
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }
}
```

{% endtab %} {% endtabs %}

- @ResponseStatus로 원하는 상태 코드를 응답할 수 있다.

{% tabs %} {% tab title="ApiExceptionV2Controller.java" %}

```java

@Slf4j
@RestController
public class ApiExceptionV2Controller {

    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandle(UserException e) {
        log.error("[exceptionHandle] ex", e);
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }
}
```

{% endtab %} {% endtabs %}

- @ExceptionHandler 안에 있던 예외를 생략하고 파라미터에 넣어 지정할 수도 있다.

{% tabs %} {% tab title="ApiExceptionV2Controller.java" %}

```java

@Slf4j
@RestController
public class ApiExceptionV2Controller {

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandle(Exception e) {
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("EX", "내부 오류");
    }
}
```

{% endtab %} {% endtabs %}

- @ExceptionHandler는 해당 예외의 자식까지 잡는다.
- 더 자세한 것이 우선권을 잡기 때문에, 부모와 자식 둘 다 있으면 자식이 호출된다.
    - 따라서 UserException이 발생하면 UserException이 정의된 메서드가 그대로 호출된다.

```java

@Slf4j
@RestController
public class Controller {

    @ExceptionHandler({AException.class, BException.class})
    public String ex(Exception e) {
        log.info("exception e", e);
    }
}
```

- 여러 예외를 한 번에 처리할 수도 있다.

### 파라미터와 응답

```java

@Slf4j
@RestController
public class Controller {
    
    @ExceptionHandler(ViewException.class)
    public ModelAndView ex(ViewException e) {
        log.info("exception e", e);
        return new ModelAndView("error");
    }
}
```

- 스프링 컨트롤러의 응답처럼 다양한 파라미터와 응답을 지정할 수 있다.

[@ExceptionHandler의 파라미터와 응답](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler-args)
