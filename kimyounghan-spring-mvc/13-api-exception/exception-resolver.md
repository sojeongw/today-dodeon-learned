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

- HandlerExceptionResolver를 직접 사용하는 건 복잡하다.
    - API 요청 오류는 response에 직접 데이터를 넣어야 해서 불편하다.
    - ModelAndView를 반환하는 것도 API 요청 형식에 맞지 않는다.
- 스프링은 이 문제를 해결하기 위해 @ExceptionHandler를 제공한다.
