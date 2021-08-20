# HTTP 요청의 기본 및 헤더 조회

```java

@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(
            HttpServletRequest request,
            HttpServletResponse response,
            HttpMethod httpMethod,
            Locale locale,
            // header 한 번에 다 받기
            @RequestHeader MultiValueMap<String, String> headerMap,
            // 필요한 header를 key 값으로 해서 받기
            @RequestHeader("host") String host,
            @CookieValue(value = "myCookie", required = false) String cookie
    ) {
        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("header host={}", host);
        log.info("myCookie={}", cookie);

        return "ok";
    }
}
```

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-02-28%20오후%2011.42.26.png)

- header 값을 간단하게 가져올 수 있다.

```java
class Example {
    void example() {
        MultiValueMap<String, String> map = new LinkedMultiValueMap();
        map.add("keyA", "value1");
        map.add("keyA", "value2");

        // [value1, value2]
        List<String> values = map.get("keyA");
    }
}
```

- MultiValueMap
    - 하나의 키에 여러 값을 받을 수 있다.
    - HTTP header, HTTP 쿼리 파라미터 처럼 키 하나에 여러 값이 있을 때 사용한다.
        - keyA=value1&keyA=value2

### Reference

[@Controller에서 사용 가능한 파라미터 목록](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)
[@Controller에서 사용 가능한 응답 값 목록](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)