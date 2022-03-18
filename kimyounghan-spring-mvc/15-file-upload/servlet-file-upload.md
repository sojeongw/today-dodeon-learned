# 서블릿과 파일 업로드

{% tabs %} {% tab title="ServletUploadControllerV1.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/servlet/v1")
public class ServletUploadControllerV1 {

    @GetMapping("/upload")
    public String newFile() {
        return "upload-form";
    }

    @PostMapping("/upload")
    public String saveFileV1(HttpServletRequest request) throws ServletException, IOException {
        log.info("request={}", request);

        String itemName = request.getParameter("itemName");
        log.info("itemName={}", itemName);

        // multipart/form-data 전송 방식에서 각각의 데이터를 받을 수 있다.
        Collection<Part> parts = request.getParts();
        log.info("parts={}", parts);

        return "upload-form";
    }

}
```

{% endtab %} {% endtabs %}

- 서블릿 요청에서 multipart/form-data로 보낸 각각의 데이터를 추출할 수 있다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/15/screenshot%202022-03-27%20오전%2011.36.34.png)

상품명과 파일을 제출하면

![](../../.gitbook/assets/kimyounghan-spring-mvc/15/screenshot%202022-03-27%20오전%2011.34.24.png)

HTTP 요청 메시지에 2개의 데이터가 서로 분리되어 들어온 걸 확인할 수 있다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/15/screenshot%202022-03-27%20오전%2011.35.33.png)

로그를 보면 parts에 데이터가 2개다.

## multipart 사용 옵션

### 업로드 사이즈 제한

```properties
spring.servlet.multipart.max-file-size=1MB
spring.servlet.multipart.max-request-size=10MB
```

- 지정한 사이즈를 넘으면 SizeLimitExceededException이 발생한다.
- max-file-size
    - 파일 하나의 최대 사이즈
    - 기본값은 1MB
- max-request-size
    - multipart 요청 하나에 여러 파일을 업로드 할 때 그 전체 파일 사이즈
    - 기본값은 10MB

### spring.servlet.multipart.enabled

```properties
spring.servlet.multipart.enabled=false
```

- 옵션을 false로 두면 서블릿 컨테이너가 multipart 요청은 처리하지 않는다.

```text
request=org.apache.catalina.connector.RequestFacade@xxx
itemName=null
parts=[]
```

- HttpServletRequest의 기본 구현체가 톰캣의 경우 RequestFacade인데 이것 그대로 들어온다.
- 요청을 찍어 보면 들어온 데이터가 없다.

```properties
spring.servlet.multipart.enabled=true
```

- 기본값
- true로 두면 서블릿 컨테이너에게 multipart 데이터를 처리하라고 지시한다.

```text
request=org.springframework.web.multipart.support.StandardMultipartHttpServletRequest
itemName=Spring
parts=[ApplicationPart1, ApplicationPart2]
```

- HttpServletRequest의 구현체가 StandardMultipartHttpServletRequest로 변해있다.
- parts에 데이터가 정상적으로 포함되었다.

### 참고

1. spring.servlet.multipart.enabled를 켜면
    - DispatcherServlet에서 MultipartResolver를 실행한다.
2. MultipartResolver는 multipart 요청이 들어오면
    - 서블릿 컨테이너가 일반적으로 전달하는 HttpServletRequest를 MultipartHttpServletRequest로 변환해서 반환한다.
    - MultipartHttpServletRequest
        - HttpServletRequest의 자식 인터페이스
        - multipart 관련 추가 기능을 제공한다.
3. 스프링 기본 multipart 리졸버는 MultipartHttpServletRequest를 구현한 StandardMultipartHttpServletRequest를 반환한다.
4. 컨트롤러는 HttpServletRequest 대신 MultipartHttpServletRequest를 주입 받게 된다.
    - multipart와 관련된 여러 처리를 편리하게 할 수 있다.
    - 하지만 추후 설명할 MultipartFile이 더 편하기 때문에 잘 사용하지 않는다.

## 프로젝트 적용

{% tabs %} {% tab title="application.properties" %}

```properties
file.dir=/Users/myMac/Downloads/
```

{% endtab %} {% tab title="ServletUploadControllerV2.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/servlet/v2")
public class ServletUploadControllerV2 {

    // properties 값을 바로 가져올 수 있다.
    @Value("${file.dir}")
    private String fileDir;

    @GetMapping("/upload")
    public String newFile() {
        return "upload-form";
    }

    @PostMapping("/upload")
    public String saveFileV2(HttpServletRequest request) throws ServletException, IOException {
        log.info("request={}", request);

        String itemName = request.getParameter("itemName");
        log.info("itemName={}", itemName);

        Collection<Part> parts = request.getParts();
        log.info("parts={}", parts);

        for (Part part : parts) {
            log.info("=== PART ===");
            log.info("name = {}", part.getName());

            // part도 헤더와 바디로 나눠져있다.
            Collection<String> headerNames = part.getHeaderNames();

            for (String headerName : headerNames) {
                log.info("header {}: {}", headerName, part.getHeader(headerName));
            }

            // content-disposition의 fileName
            log.info("submittedFileName = {}", part.getSubmittedFileName());
            log.info("size = {}", part.getSize());

            // 데이터를 읽는다.
            InputStream inputStream = part.getInputStream();
            String body = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
            log.info("body = {}", body);

            // 파일에 저장한다.
            if (StringUtils.hasText(part.getSubmittedFileName())) {
                String fullPath = fileDir + part.getSubmittedFileName();
                log.info("파일 저장 fullPath = {}", fullPath);

                part.write(fullPath);
            }
        }

        return "upload-form";
    }

}
```

{% endtab %} {% endtabs %}

- application.properties에 파일을 저장할 경로를 지정한다.
- @Value
    - properties에 있는 경로 주입
- getSubmittedFileName()
    - 클라이언트가 전달한 파일명
- getInputStream()
    - Part의 전송 데이터 조회
- write()
    - Part를 통해 전송된 데이터 저장

![](../../.gitbook/assets/kimyounghan-spring-mvc/15/screenshot%202022-03-27%20오후%201.23.37.png)

관련 헤더 값을 모두 조회할 수 있다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/15/screenshot%202022-03-27%20오후%201.20.34.png)

설정한 경로에 파일이 저장되었다.

### 참고

```properties
logging.level.org.apache.coyote.http11=debug
```

- 큰 파일을 업로드 할때는 로그가 너무 많이 남기 때문에 debug 옵션을 꺼둔다.

```text
log.info("body={}", body);
```

- 파일의 바이너리 데이터를 모두 출력하므로 역시 꺼두는 게 좋다.