# 서블릿

톰캣 등 WAS를 직접 설치하고 그 위에 서블릿 코드를 클래스 파일로 빌드한 뒤 톰캣 서버를 실행하면 된다. 하지만 이 과정은 매우 번거롭다.

스프링 부트는 톰캣 서버를 내장하고 있어 별도의 설치 없이 편리하게 서블릿 코드를 실행할 수 있다.

## 서블릿 등록

```java

@ServletComponentScan
@SpringBootApplication
public class ServletApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServletApplication.class, args);
    }

}
```

- `@ServletComponentScan`
  - 애너테이션이 붙은 클래스부터 하위에 있는 클래스까지 서블릿을 전부 찾아 등록해준다.

## 서블릿 구현

- `@WebServlet`
    - 서블릿 애너테이션
    - name: 서블릿 이름
    - urlPatterns: URL 매핑
    - 옵션들은 값이 겹치면 안된다.

```java
// hello로 요청이 오면 이게 실행된다.
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    // ctrl + o로 protected(자물쇠 표시)인 메서드를 오버라이드 한다.
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("HelloServlet.service");
        System.out.println("request = " + request);
        System.out.println("response = " + response);
    }
}

```

`/hello`로 요청을 보내면 웹 브라우저가 http 요청 메시지를 만들어 서버에 던지고, 서버는 응답을 위한 응답을 만들어 보낸다.

```text
HelloServlet.service
request = org.apache.catalina.connector.RequestFacade@47b556db
response = org.apache.catalina.connector.ResponseFacade@4fee490e
```

HttpServletRequest/Response는 인터페이스다. 다양한 WAS들이 이 표준 스펙을 구현하는 방식이다. 그 구현체가 Request/ResponseFacade로 찍히는 것이다.

```java

@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        System.out.println("username = " + username);
    }
}
```

```text
username = kim
```

서블릿은 쿼리 파라미터를 손쉽게 가져올 수 있도록 지원해준다.

```java

@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // header의 contentType에 들어가는 데이터
        response.setContentType("text/plain");
        response.setCharacterEncoding("utf-8");
        // body 내용 설정
        response.getWriter().write("hello " + username);
    }
}

```

![](../../.gitbook/assets/kimyounghan-spring-mvc/02/screenshot%202021-06-12%20오후%208.10.25.png)

![](../../.gitbook/assets/kimyounghan-spring-mvc/02/screenshot%202021-06-12%20오후%208.12.32.png)

response의 헤더와 바디를 설정할 수 있다.

## HTTP 요청 메시지를 로그로 확인하기

```yaml
logging.level.org.apache.coyote.http11=debug
```

`application.yaml`에 설정을 추가한다. 운영 서버에 모든 요청 정보를 남기면 성능 저하가 발생하므로 개발 단계에서만 사용하자.

## 서블릿 컨테이너 동작 방식
### 내장 톰캣 서버 생성

![](../../.gitbook/assets/kimyounghan-spring-mvc/02/screenshot%202021-06-12%20오후%208.17.47.png)

스프링 부트를 실행하면서 내장 톰캣 서버를 띄운다. 톰캣 서버는 내부에 서블릿 컨테이너를 가지고 있다. 이 서블릿 컨테이너를 통해 서블릿을 생성한다. 

### HTTP 요청, 응답 메시지

![](../../.gitbook/assets/kimyounghan-spring-mvc/02/screenshot%202021-06-12%20오후%208.17.54.png)

웹 브라우저가 요청을 생성해서 서버에 던진다.

### 웹 애플리케이션 서버의 요청 응답 구조

![](../../.gitbook/assets/kimyounghan-spring-mvc/02/screenshot%202021-06-12%20오후%208.18.00.png)

서버는 request, response 객체를 만들어 싱글턴으로 떠 있는 helloServlet를 호출한 뒤, 필요한 작업을 해서 response를 생성한다.

response content-length는 WAS가 자동으로 만들어준다.