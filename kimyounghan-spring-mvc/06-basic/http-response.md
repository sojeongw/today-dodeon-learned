# HTTP 응답

## 정적 리소스

- 웹 브라우저에 정적인 HTML, css, js를 제공할 때는 정적 리소르를 사용한다.
- 클래스 패스
    - 리소스를 보관하는 곳
    - 시작점은 `src/main/resources`
- 정적 리소스
    - 파일 내용 변경 없이 그대로 서비스 하는 것
- 스프링 부트는 클래스패스에서 아래의 디렉토리에 있는 정적 리소스를 사용한다.
    - /static
    - /public
    - /resources
    - /META-INF/resources

예를 들어 `src/main/resources/static/basic/hello-form.html`에 파일이 있다면 `http://localhost:8080/basic/hello-from.html` 로 접속할 수
있다.

## 뷰 템플릿

- 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.
- 뷰 템플릿을 거쳐 HTML이 생성되고 뷰가 응답을 만들어 전달한다.
- 스프링의 기보 뷰 템플릿 경로
    - `src/main/resources/templates`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<!-- controller에서 받아오는 데이터 -->
<p th:text="${data}">empty</p>
</body>
</html>
```

```java

@Controller
public class ResponseViewController {

    @RequestMapping("/response-view-v1")
    public ModelAndView responseViewV1() {
        ModelAndView mav = new ModelAndView("response/hello").addObject("data", "hello!");
        return mav;
    }
}
```

- 실행하면 html에서 `hello!`라는 데이터를 불러와 출력한다.

```java

@Controller
public class ResponseViewController {

    @RequestMapping("/response-view-v2")
    public String responseViewV2(Model model) {
        model.addAttribute("data", "hello!!");
        return "response/hello";
    }
}
```

- @Controller에서 String을 반환하면 뷰의 논리 이름으로 인식하고 해당 뷰를 찾아 반환한다.
- @ResponseBody가 붙었다면 그냥 `hello!!`라는 문자열만 출력할 것이다.

```java

@Controller
public class ResponseViewController {

    @RequestMapping("/response/hello")
    public void responseViewV3(Model model) {
        model.addAttribute("data", "hello!!");
    }
}
```

- 뷰의 위치와 매핑된 path가 같고 void면 뷰 지정을 생략할 수 있다.
    - 따라서 `hello!`가 찍히는 html이 그대로 출력된다.
    - @Controller를 사용하고 HttpServletRequestOutputStream처럼 HTTP 메시지 바디를 처리하는 파라미터가 없어야 한다.
- 명시성이 떨어지고 이렇게 파일 경로와 딱 맞아 떨어지는 상황이 별로 없기 때문에 권장하지 않는다.

### 타임리프 기본 설정

```properties
# 기본 값이라 적지 않아도 이렇게 들어간다.
# 따로 설정하고 싶을 때 이 값을 바꿔주면 된다.
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

## HTTP 메시지

- HTTP API를 제공하는 경우엔 HTML 대신 데이터를 전달해야 하므로 HTTP 메시지 바디에 JSON 등을 보낸다.
- HTML이나 뷰 템플릿도 HTTP 응답 메시지 바디에 담겨서 전달되지만, 여기서는 정적 리소스나 뷰 템플릿 없이 직접 HTTP 응답 메시지를 전달하는 경우를 말한다.

### 단순 텍스트

```java

@Slf4j
@Controller
public class ResponseBodyController {

    @GetMapping("/response-body-string-v1")
    public void responseBodyV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");
    }
}
```

- 바디에 직접 ok를 담아 보낸다.

```java

@Slf4j
@Controller
public class ResponseBodyController {

    @GetMapping("/response-body-string-v2")
    public ResponseEntity<String> responseBodyV2() {
        return new ResponseEntity<>("ok", HttpStatus.OK);
    }
}
```

- ResponseEntity
    - HttpEntity를 상속받았기 때문에 HTTP 메시지의 헤더, 바디 정보를 가지고 있다.
    - ResponseEntity는 HTTP 응답 코드까지 설정할 수 있다.

```java

@Slf4j
@Controller
public class ResponseBodyController {

    @ResponseBody
    @GetMapping("/response-body-string-v3")
    public String responseBodyV3() {
        return "ok";
    }
}
```

- @ResponseBody
    - 뷰를 사용하지 않고 HTTP 메시지 컨버터를 통해 직접 HTTP 메시지를 입력할 수 있다.

### JSON

```java

@Slf4j
@Controller
public class ResponseBodyController {

    @GetMapping("/response-body-json-v1")
    public ResponseEntity<HelloData> responseBodyJsonV1() {
        HelloData helloData = new HelloData();

        helloData.setUsername("userA");
        helloData.setAge(20);

        return new ResponseEntity<>(helloData, HttpStatus.OK);
    }
}
```

- ResponseEntity에 HttpStatus를 담아 보낸다.

```java

@Slf4j
@Controller
public class ResponseBodyController {

    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    @GetMapping("/response-body-json-v2")
    public HelloData responseBodyJsonV2() {
        HelloData helloData = new HelloData();

        helloData.setUsername("userA");
        helloData.setAge(20);

        return helloData;
    }
}
```

- ResponseEntity 없이 상태 코드를 반환하고 싶다면 @ResponseStatus를 사용한다.
    - 애너테이션이기 때문에 응답 코드를 동적으로 설정할 수 없다.
    - 조건에 따라 동적으로 하려면 ResponseEntity를 사용한다.

```java

@ResponseBody
public class ResponseBodyController {

}
```

- @ResponseBody를 메서드마다 붙이기 귀찮으면 클래스에 달아도 된다.

```java

@RestController
public class ResponseBodyController {

}
```

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-03-01%20오후%204.40.56.png)

- @Controller와 @ResponseBody를 합친 것이 @RestController다.
- 뷰 템플릿 대신 HTTP 메시지 바디에 직접 데이터를 입력한다.
- 이름 그대로 REST API(HTTP API)를 만드는 컨트롤러다.