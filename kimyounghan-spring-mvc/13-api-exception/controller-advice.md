# Controller Advice

- @ExceptionHandler를 사용하면 일반 코드와 예외 처리 코드가 하나에 섞여있게 된다.
- Controller Advice로 이 둘을 분리해보자.

{% tabs %} {% tab title="ExControllerAdvice.java" %}

```java

@Slf4j
@RestControllerAdvice
public class ExControllerAdvice {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandle(IllegalArgumentException e) {
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }

    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandle(UserException e) {
        log.error("[exceptionHandle] ex", e);
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandle(Exception e) {
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("EX", "내부 오류");
    }
}
```

{% endtab %} {% tab title="ApiExceptionV2Controller.java" %}

```java

@Slf4j
@RestController
public class ApiExceptionV2Controller {

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
}
```

{% endtab %} {% endtabs %}

- 기존에 컨트롤러에 함께 있던 @ExceptionHandler를 ExControllerAdvice로 분리한다.
- ExControllerAdvice에는 @RestControllerAdvice를 적용한다.
- 실행하면 이전과 똑같이 동작하는 걸 알 수 있다.

## @ControllerAdvice

- 대상으로 지정한 여러 컨트롤러에 @ExceptionHandler, @InitBinder 기능을 부여한다.
- 예시처럼 대상을 지정하지 않으면 모든 컨트롤러에 적용된다.
- @RestControllerAdvice
    - @ControllerAdvice 기능에 @ResponseBody가 추가되어 있다.

## 대상 컨트롤러 지정

```java
// @RestController가 붙은 모든 컨트롤러
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {
}

// 특정 패키지에 있는 모든 컨트롤러
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {
}

// 명시한 클래스만 적용
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {
}
```

- 자식 클래스가 있다면 똑같이 적용된다.
- 대상을 지정하지 않으면 모든 컨트롤러에 적용된다.

[컨트롤러 지정 방법](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice)