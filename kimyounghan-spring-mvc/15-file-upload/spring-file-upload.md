# 스프링과 파일 업로드

- 스프링은 이전의 복잡한 과정을 MultipartfIle이라는 인터페이스로 지원한다.

{% tabs %} {% tab title="SpringUploadController.java" %}

```java
@Slf4j
@Controller
@RequestMapping("/spring")
public class SpringUploadController {

    @Value("${file.dir}")
    private String fileDir;

    @GetMapping("/upload")
    public String newFile() {
        return "upload-form";
    }

    @PostMapping("/upload")
    // HttpServletRequest 대신 데이터를 바로 매핑해서 가져올 수 있다.
    public String saveFile(@RequestParam String itemName,
                           // 스프링이 지원하는 인터페이스
                           @RequestParam MultipartFile file,
                           HttpServletRequest request) throws IOException {
        log.info("request={}", request);
        log.info("itemName={}", itemName);
        log.info("multipartFile={}", file);

        if (!file.isEmpty()) {
            String fullPath = fileDir + file.getOriginalFilename();
            log.info("파일 저장 fullPath={}", fullPath);

            file.transferTo(new File(fullPath));
        }
        return "upload-form";
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/15/screenshot%202022-03-27%20오후%201.35.16.png)

- ArgumentResolver 덕에 서블릿 대신 직접 데이터를 가져올 수 있게 되었다.
- 파일 정보를 복잡한 과정 없이 가져올 수 있다.