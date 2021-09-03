# HTTP 요청 메시지

## 단순 텍스트

- HTTP message body에 직접 데이터를 담아 요청할 수 있다.
- HTTP API에서 주로 사용한다.
    - JSON, XML, TEXT
- POST, PUT, PATCH에 사용한다.
- 요청 파라미터(GET 쿼리 스트링, POST HTML Form)는 @RequestParam, @ModelAttribute를 쓸 수 있지만 그 외는 불가하다.
    - HTML Form 중에 메시지 바디에 담겨져 오는 경우는 가능하다.

### InputStream

```java

@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest request,
                                  HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        // Stream은 바이트 코드이기 때문에 항상 뭘로 인코딩할지 정해야 한다.
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody = {}", messageBody);
        response.getWriter().write("ok");
    }
}
```

- HttpServletRequest에서 InputStream을 가져올 수 있다.

```java

@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v2")
    public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody = {}", messageBody);
        responseWriter.write("ok");
    }
}
```

[@Controller에서 사용 가능한 파라미터 목록](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)

- 문서를 보면 InputStream과 Writer를 직접 가져올 수 있어 코드가 간략해진다.
- InputStream
    - HTTP 요청 메시지 바디의 내용을 직접 조회한다.
- OutputStream
    - HTTP 응답 메시지 바디에 결과를 직접 출력한다.

### HttpEntity

```java

@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
        // 스프링이 알아서 문자로 컨버팅 해준다.
        String messageBody = httpEntity.getBody();

        log.info("messageBody = {}", messageBody);
        // HTTP 바디 정보를 직접 반환해준다.
        return new HttpEntity<>("ok");
    }
}
```

- HttpEntity
    - HTTP header, body를 편리하게 조회할 수 있다.
    - 요청 파라미터를 조회하는 기능과는 아무 관계가 없다.
        - @RequestParam, @ModelAttribute
        - 이 둘은 요청 파라미터인 쿼리 파라미터나 HTML Form 방식이라는 걸 잊지 말자.
- 응답에도 사용 가능하다.
    - 메시지 바디 정보를 직접 반환할 수 있다.
    - 헤더 정보를 포함할 수 있다.
    - view 조회는 불가하다.
- 메시지 컨버터를 통해 HTTP 바디를 문자나 객체로 변환한다.

### HttpEntity를 상속받은 객체들

- RequestEntity
- ResponseEntity

```text
return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)
```

HTTP 상태 코드를 넣어줄 수 있다.

### @RequestBody, ResponseBody

```java

@Slf4j
@Controller
public class RequestBodyStringController {

    @ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody) {
        log.info("messageBody = {}", messageBody);
        return "ok";
    }
}
```

- @RequestBody
    - HttpEntity 대신 더 간단하게 사용할 수 있다.
    - 헤더 정보가 필요하면 HttpEntity나 @RequestHeader를 사용하면 된다.
- @ResponseBody
    - HTTP 메시지 바디에 응답을 직접 담아 전달할 수 있다.
- 물론 둘 다 view는 사용하지 않는다.

### 요청 파라미터 vs HTTP 메시지 바디

다시 한 번 강조하지만, 이렇게 메시지 바디를 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam, @ModelAttribute와는 전혀 관계가 없다.

- 요청 파라미터 조회
    - @RequestParam
- HTTP 메시지 바디 직접 조회
    - @RequestBody

## JSON

