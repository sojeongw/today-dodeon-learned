# HTTP 요청 파라미터

클라이언트에서 서버로 데이터를 보낼 때는 주로 3가지 방법을 사용한다.

- GET
    - 쿼리 파라미터
        - /url?username=hello&age=20
    - 메시지 바디 없이 URL 쿼리 파라미터에 데이터를 포함해 전달한다.
    - 검색, 필터, 페이징 등에서 사용한다.
- POST
    - HTML Form
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파라미터 형식으로 전달한다.
        - username=hello&age=20
    - 회원 가입, 상품 주문, HTML Form에 사용한다.
- HTTP message body에 직접 담아서 요청
    - HTTP API에 주로 사용한다.
        - json, xml, text
        - 주로 json 데이터를 사용한다.
    - POST, PUT, PATCH에 쓰인다.

## 쿼리 파라미터, HTML Form

```text
# GET 쿼리 파라미터
http://localhost:8080/request-param?username=hello&age=20

# POST HTML Form 전송
POST /request-param ...
content-type: application/x-www-form-urlencoded
username=hello&age=20
```

- 요청 파랄미터 조회라고도 한다.
- HttpServletRequest의 request.getParameter()
    - GET 쿼리 파라미터든 POST HTML Form이든 둘 다 형식이 같으므로 구분없이 조회할 수 있다.

```java

@Slf4j
@Controller
public class RequestParamController {

    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        log.info("username = {}, age = {}", username, age);

        response.getWriter().write("ok");
    }
}
```

- request.getParameter()로 요청 파라미터를 조회한다.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/request-param-v1" method="post">
    username: <input type="text" name="username"/> age: <input type="text" name="age"/>
    <button type="submit">전송</button>
</form>
</body>
</html>
```

```text
2022-03-01 00:10:55.961  INFO 23155 --- [nio-8080-exec-2] h.s.b.request.RequestParamController     : username = 도던, age = 33
```

- HTML Form으로 요청 파라미터를 조회한다.