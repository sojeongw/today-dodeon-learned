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

### HttpServletRequest

```java

@Slf4j
@Controller
public class RequestBodyJsonController {

    private ObjectMapper objectMapper = new ObjectMapper();

    @PostMapping("/request-body-json-v1")
    public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        log.info("messageBody={}", messageBody);

        HelloData data = objectMapper.readValue(messageBody, HelloData.class);
        log.info("username={}, age={}", data.getUsername(), data.getAge());

        response.getWriter().write("ok");
    }
}
```

- Jackson 라이브러리인 objectMapper로 JSON 데이터를 컨버팅한다.

### @RequestBody

```java

@Slf4j
@Controller
public class RequestBodyJsonController {

    private ObjectMapper objectMapper = new ObjectMapper();

    @ResponseBody
    @PostMapping("/request-body-json-v2")
    public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
        HelloData data = objectMapper.readValue(messageBody, HelloData.class);

        log.info("username={}, age={}", data.getUsername(), data.getAge());
        return "ok";
    }
}
```

- @RequestBody로 InputStream을 사용하는 번거로운 절차를 생략했다.
- 여전히 objectMapper로 굳이 바꿔줘야 한다.

```java

@Slf4j
@Controller
public class RequestBodyJsonController {

    private ObjectMapper objectMapper = new ObjectMapper();

    @ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData data) {
        log.info("username={}, age={}", data.getUsername(), data.getAge());
        return "ok";
    }
}
```

- String 대신 객체 자체로 바로 받을 수 있다.
- HTTP 메시지 컨버터가 HTTP 메시지 바디를 문자나 객체, JSON으로 바꿔준다.
- @RequestBody는 생략할 수 없다.
    - 생략 시 단순한 타입 외에는 모두 @ModelAttribute를 적용하도록 되어있기 때문이다.
    - 따라서 메시지 바디가 아니라 요청 파라미터를 처리하게 된다.

### ResponseBody

```java

@Slf4j
@Controller
public class RequestBodyJsonController {
    @ResponseBody
    @PostMapping("/request-body-json-v5")
    public HelloData requestBodyJsonV5(@RequestBody HelloData data) {
        log.info("username={}, age={}", data.getUsername(), data.getAge());

        // 객체를 자동으로 JSON으로 반환해준다.
        return data;
    }
}
```

- @RequestBody
    - JSON 요청 -> HTTP 메시지 컨버터 -> 객체
    - `content-type:application/json`
- @ResponseBody
    - 객체 -> HTTP 메시지 컨버터 -> JSON 응답
    - `accept:application/json`

참고로, content-type이 application/json이어야 JSON을 사용할 수 있는 메시지 컨버터가 실행된다.

### HttpEntity

```java

@Slf4j
@Controller
public class RequestBodyJsonController {

    private ObjectMapper objectMapper = new ObjectMapper();

    @ResponseBody
    @PostMapping("/request-body-json-v4")
    public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) {
        HelloData data = httpEntity.getBody();

        log.info("username={}, age={}", data.getUsername(), data.getAge());
        return "ok";
    }
}
```

- 단순 텍스트 때 배운 것과 마찬가지로 HttpEntity도 사용 가능하다.