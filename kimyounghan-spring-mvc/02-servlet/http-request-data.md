# HTTP 요청 데이터

HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터를 전달하는 방법을 알아보자.

## GET 쿼리 파라미터

```text
/url?username=hello&age=20
```

- 메시지 바디 없이 URL의 쿼리 파라미터에 데이터를 포함해서 전달한다. 
- 검색, 필터, 페이징 등에서 많이 사용하는 방식이다.

```java
@WebServlet(name = "requestParamServlet", urlPatterns = "/request-param")
public class RequestParamServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("[전체 파라미터 조회]");
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> System.out.println(paramName +
                        "=" + request.getParameter(paramName)));

        // 단일 파라미터 조회
        System.out.println("[단일 파라미터 조회]");
        String username = request.getParameter("username");
        System.out.println("request.getParameter(username) = " + username);

        String age = request.getParameter("age");
        System.out.println("request.getParameter(age) = " + age);
        System.out.println();

        // 파라미터 이름 모두 조회
        Enumeration<String> parameterNames = request.getParameterNames();

        // 파라미터를 Map으로 조회
        Map<String, String[]> parameterMap = request.getParameterMap();

        // 복수의 파라미터 조회
        System.out.println("[이름이 같은 복수 파라미터 조회]");
        System.out.println("request.getParameterValues(username)");
        String[] usernames = request.getParameterValues("username");
        for (String name : usernames) {
            System.out.println("username=" + name);
        }

        response.getWriter().write("ok");
    }
}
```

```text
http://localhost:8080/request-param?username=dodeon&age=20&username=sojeong
```

```text
[전체 파라미터 조회]
username=dodeon
age=20

[단일 파라미터 조회]
request.getParameter(username) = dodeon
request.getParameter(age) = 20

[이름이 같은 복수 파라미터 조회]
request.getParameterValues(username)
username=dodeon
username=sojeong
```

### 복수 파라미터에서 단일 파라미터 조회

파라미터 이름은 하나인데 값이 중복이라면 request.getParameterValues()를 사용해야 한다.

request.getParameter()는 단 하나의 값만 있을 때 사용한다. 중복일 때 사용하면 request.getParameterValues()의 첫번째 값을 반환한다. 

## POST HTML Form

```text
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

username=kim&age=20
```

- 메시지 바디에 쿼리 파라미터 형식으로 전달한다.
  - username=kim&age=20
- 회원 가입, 상품 주문 등에 사용한다.

쿼리 파라미터 조회 메서드를 그대로 사용한다. 클라이언트 입장에서는 GET과 POST HTML Form이 차이가 있지만 서버 입장에서는 둘의 형식이 동일하므로 request.getParameter()로 구분없이 편리하게 조회할 수 있다.

즉, request.getParameter()는 GET URL 쿼리 파라미터와 POST HTML Form 형식 둘 다 지원한다.

### 참고

content-type은 HTTP 메시지 바디의 데이터 형식을 지정한다.

- GET URL 쿼리 파라미터 형식으로 전달할 때
  - HTTP 메시지 바디를 사용하지 않기 때문에 content-type이 없다.
- POST HTML Form 형식으로 전달하 때
  - HTTP 메시지 바디에 해당 데이터를 포함해서 보낸다.
  - 따라서 바디에 포함된 데이터가 어떤 형식인지 content-type을 꼭 지정해야 한다. 
  - 이렇게 폼으로 데이터를 전송하는 형식을 application/x-www-form-urlencoded라고 한다.
  
### Postman 테스트 방법

- POST 전송
- Headers에 content-type: application/x-www-form-urlencoded 추가
- Body에 x-www-form-urlencoded 선택

## HTTP message body

- 메시지 바디에 직접 담아서 요청한다.
- HTTP API에서 주로 사용한다.
- json, xml, text 
- 주로 json을 사용한다.
- POST, PUT, PATCH에서 사용한다.