# HttpServletResponse

- HTTP 응답 메시지를 생성하는 역할을 한다.
- HTTP 응답 코드를 지정한다.
- 헤더와 바디를 생성한다.
- 편의 기능을 제공한다.
    - content-type, 쿠키, redirect

```java
package hello.servlet.basic.response;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")
public class ResponseHeaderServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // status
        response.setStatus(HttpServletResponse.SC_OK);

        // headers
        response.setHeader("Content-Type", "text/plain;charset=utf-8");
        response.setHeader("Cache-Control", "no-cache, no-store, must-revalidate");
        response.setHeader("Pragma", "no-cache");
        // 임의의 헤더 생성 가능
        response.setHeader("my-header", "hello");

        // 편의 메서드
        content(response);
        cookie(response);
        redirect(response);

        // message body
        PrintWriter writer = response.getWriter();
        writer.print("ok");
    }

    private void content(HttpServletResponse response) {
        /*
        Content-Type: text/plain;charset=utf-8
        Content-Length: 2
        response.setHeader("Content-Type", "text/plain;charset=utf-8");
        */
        response.setHeader("Content-Type", "text/plain;charset=utf-8");
        response.setContentType("text/plain");
        response.setCharacterEncoding("utf-8");
        // 생략 시 자동 생성
        // response.setContentLength(2);
    }

    private void cookie(HttpServletResponse response) {
        /*
        Set-Cookie: myCookie=good; Max-Age=600;
        response.setHeader("Set-Cookie", "myCookie=good; Max-Age=600");
        */
        Cookie cookie = new Cookie("myCookie", "good");
        cookie.setMaxAge(600); //600초
        response.addCookie(cookie);
    }

    private void redirect(HttpServletResponse response) throws IOException {
        /*
        Status Code 302
        Location: /basic/hello-form.html
        */
        response.setStatus(HttpServletResponse.SC_FOUND);
        response.setHeader("Location", "/basic/hello-form.html");
        response.sendRedirect("/basic/hello-form.html");
    }
}

```