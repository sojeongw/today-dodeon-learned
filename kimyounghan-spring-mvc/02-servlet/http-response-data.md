# HTTP 응답 데이터

## 텍스트 응답

```java

@WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")
public class ResponseHeaderServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        PrintWriter writer = response.getWriter();
        writer.print("ok");
    }
}
```

## HTML 응답

```java
@WebServlet(name = "responseHtmlServlet", urlPatterns = "/response-html")
public class ResponseHtmlServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // Content-Type: text/html;charset=utf-8
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        
        PrintWriter writer = response.getWriter(); writer.println("<html>"); writer.println("<body>");
        writer.println(" <div>안녕?</div>"); writer.println("</body>"); writer.println("</html>");
    }
}
```

```html
<html>

<body>
<div>안녕?</div>
</body>

</html>
```

- HTML을 반환할 때는 content-type을 text/html로 지정해야 한다.

## API JSON

```java
@WebServlet(name = "responseJsonServlet", urlPatterns = "/response-json")
public class ResponseJsonServlet extends HttpServlet {
    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // Content-Type: application/json
        response.setHeader("content-type", "application/json");
        response.setCharacterEncoding("utf-8");

        HelloData data = new HelloData();
        data.setUsername("kim");
        data.setAge(20);

        // {"username":"kim","age":20}
        String result = objectMapper.writeValueAsString(data);
        response.getWriter().write(result);
    }
}

```

```json
{"username":"kim","age":20}
```

- json을 반환할 때는 content-type을 application/json으로 지정해야 한다. 
- jackson이 제공하는 objectMapper.writeValueAsString()을 사용하면 객체를 json으로 파싱할 수 있다.

### 참고

application/json은 utf-8을 사용하도록 정의되어 있다. 그래서 `charset=utf-8`같은 추가 파라미터를 지원하지 않으며 `application/json;charset=utf-8`이라고 전달하는 건 의미 없는 파라미터다.

그런데 위의 코드를 실행하면 utf-8이 같이 출력된다. response.getWriter()를 사용하면 추가 파라미터를 자동으로 추가하기 때문이다. response.getOutputStream()으로 출력하면 그런 문제가 없다.