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

