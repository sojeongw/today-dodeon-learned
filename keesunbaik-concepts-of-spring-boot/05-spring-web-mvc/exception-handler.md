# ExceptionHandler

- BasicErrorController
    - 스프링 부트가 제공하는 기본 예외 처리기
    - 기본적으로 ExceptionHandler를 등록해 동작한다.
    - HTML, JSON 응답을 지원한다.

{% tabs %} {% tab title="SampleController.java" %}

```java

@Controller
public class SampleController {

    @GetMapping("/exception")
    public String exception() {
        throw new SampleException();
    }

    @ExceptionHandler(SampleException.class)
    public @ResponseBody AppError sampleError(SampleException e) {
        AppError appError = new AppError();
        appError.setMessage("error.app.key");
        appError.setReason("IDK");

        return appError;
    }
}
```

{% endtab %} {% tab title="AppError.java" %}

```java
public class AppError {
    private String message;
    private String reason;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getReason() {
        return reason;
    }

    public void setReason(String reason) {
        this.reason = reason;
    }
}
```

{% endtab %}{% tab title="SampleException.java" %}

```java
public class SampleException extends RuntimeException {
}

```

{% endtab %} {% endtabs %}

```json
{
  "message": "error.app.key",
  "reason": "IDK"
}
```

- SampleException이 발생하면 ExceptionHandler가 실행된다.

## 전역적으로 적용하기

- @ControllerAdvice
- @ExchangeHandler

## 커스텀 에러 페이지

### 상태 코드 값에 따라 에러 페이지 보여주기

- src/main/resources/static|template/error/
    - html 이름이 상태 코드 값과 동일하거나 5xx와 같아야 한다.
        - 404.html
        - 5xx.html
- 더 많은 커스터마이징이 필요하면 ErrorViewResolver를 구현한다.
    - 동적으로 더 다양한 옵션을 선택할 수 있다.