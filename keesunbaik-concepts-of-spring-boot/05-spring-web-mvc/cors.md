# CORS

## SOP

- Single Origin Policy
- 같은 오리진에만 요청을 보낼 수 있다.

## CORS

- Cross Origin Resource Sharing
- 서로 다른 오리진끼리 리소스를 공유할 수 있다.

## Origin?

- URI 스키마
    - http, https
- hostname
    - localhost
    - whiteship.me
- 포트
    - 8080
    - 18080
- 기본적으로는 SOP가 적용되어 있어 오리진이 다르면 서로 호출할 수 없다.

## @CrossOrigin

- 스프링 MVC가 지원한다.
- 스프링 부트가 여러 빈 설정을 자동으로 해줘서 쉽게 사용할 수 있다.
    - @Controller나 @RequestMapping에 추가한다.
    - WebMvcConfigurer를 사용해 글로벌로 설정한다.

```java

@RestController
public class Application {

    @CrossOrigin(origins = "http://localhost:18080")
    @GetMapping("/cross-origin")
    public String crossOrigin() {
        return "crossOrigin";
    }
    
}
```