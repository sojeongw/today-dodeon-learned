# HttpServletRequest

HTTP 요청 메시지를 개발자가 직접 파싱하는 것은 불편하다. 서블릿은 개발자 대신에 HTTP 요청 메시지를 파싱한다. 그리고 그 결과를 HttpServletRequest 객체에 담아서 제공한다.

## HTTP 요청 메시지 조회 기능

```text
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

username=kim&age=20
```

- start line
    - http 메서드
    - url
    - 쿼리 스트링
    - 스키마, 프로토콜
- header
    - host
    - content-type
- body
    - form 파라미터 or message body 등의 데이터 조회

HttpServletRequest를 사용하면 위와 같은 HTTP 요청 메시지를 편리하게 조회할 수 있다.

## 임시 저장소 기능

- 해당 HTTP 요청의 시작부터 끝까지 유지되는 임시 저장소 기능
- 저장
    - request.setAttribute(name, value)
- 조회
    - request.getAttribute(name)

## 세션 관리 기능

- 세션을 이용해 로그인을 유지할 수 있다.
- request.getSession(create: true)

## 주의 사항

HttpServletRequest, HttpServletResponse는 HTTP 요청과 응답 메시지를 편리하게 사용할 수 있도록 도와주는 객체다. 따라서 이 기능을 깊이 이해하려면 HTTP 스펙이 제공하는 요청,
응답 메시지를 이해해야 한다.

## Start Line

```java

@WebServlet(name = "requestHeaderServlet", urlPatterns = "/request-header")
public class RequestHeaderServlet extends HttpServlet {

    // public 말고 protected된 메서드를 사용해야 한다.
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        printStartLine(request);
    }

    // start line 정보
    private void printStartLine(HttpServletRequest request) {
        System.out.println("--- REQUEST-LINE - start ---");

        // get, post 등 http 메서드
        System.out.println("request.getMethod() = " + request.getMethod());

        // HTTP/1.1
        System.out.println("request.getProtocol() = " + request.getProtocol());

        // http
        System.out.println("request.getScheme() = " + request.getScheme());

        // http://localhost:8080/request-header
        System.out.println("request.getRequestURL() = " + request.getRequestURL());

        // /request-test
        System.out.println("request.getRequestURI() = " + request.getRequestURI());

        // username=hi
        System.out.println("request.getQueryString() = " + request.getQueryString());

        // https 사용 유무
        System.out.println("request.isSecure() = " + request.isSecure());

        System.out.println("--- REQUEST-LINE - end ---\n");
    }

}
```

```text
http://localhost:8080/request-header
```

위의 url로 요청하면

```text
--- REQUEST-LINE - start ---
request.getMethod() = GET
request.getProtocol() = HTTP/1.1
request.getScheme() = http
request.getRequestURL() = http://localhost:8080/request-header
request.getRequestURI() = /request-header
request.getQueryString() = null
request.isSecure() = false
--- REQUEST-LINE - end ---
```

결과가 출력된다.

```text
http://localhost:8080/request-header?username=dodeon
```

쿼리 스트링을 넣어서 요청하면

```text
--- REQUEST-LINE - start ---
request.getMethod() = GET
request.getProtocol() = HTTP/1.1
request.getScheme() = http
request.getRequestURL() = http://localhost:8080/request-header
request.getRequestURI() = /request-header
request.getQueryString() = username=dodeon
request.isSecure() = false
--- REQUEST-LINE - end ---
```

뭐리 스트링이 찍힌다.

## Header

```java

@WebServlet(name = "requestHeaderServlet", urlPatterns = "/request-header")
public class RequestHeaderServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        printHeaders(request);
    }

    // Header 모든 정보
    private void printHeaders(HttpServletRequest request) {
        System.out.println("--- Headers - start ---");

        /* 옛날 방식
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
              String headerName = headerNames.nextElement();
            System.out.println(headerName + ": " + request.getHeader(headerName));
        }
        */

        request.getHeaderNames().asIterator()
                .forEachRemaining(headerName -> System.out.println(headerName + ":" + request.getHeader(headerName)));
        System.out.println("--- Headers - end ---\n");
    }
}
```

```text
--- Headers - start ---
host:localhost:8080
connection:keep-alive
cache-control:max-age=0
sec-ch-ua:" Not;A Brand";v="99", "Google Chrome";v="91", "Chromium";v="91"
sec-ch-ua-mobile:?0
dnt:1
upgrade-insecure-requests:1
user-agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.106 Safari/537.36
accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
sec-fetch-site:none
sec-fetch-mode:navigate
sec-fetch-user:?1
sec-fetch-dest:document
accept-encoding:gzip, deflate, br
accept-language:ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
cookie:optimizelyEndUserId=oeu1610084455381r0.7319812782071056; ch-veil-id=9f3b199b-8db0-4d26-91b6-37ba33091c86; lang=ko; _ga=GA1.1.1954188695.1620027883; Idea-d9466cd9=1bac0d08-b7da-49a6-98bb-ce36acec26a6; ajs_anonymous_id=%22b7ecbc45-f07a-47f9-b0d4-647e30513dde%22; wcs_bt=s_4288e28adaa4:1623751338; amplitude_id_e3025c63ec54f7c9aa28aa2f31de7f2c=eyJkZXZpY2VJZCI6IjM0ZjUwZmVlLWFiZGQtNDgxZC1iMDZiLWE1NDBlOTlkYzhlYVIiLCJ1c2VySWQiOiJTcGpBWVp6eEhIZGJObUVlUGpDdFhWaUprcmYyIiwib3B0T3V0IjpmYWxzZSwic2Vzc2lvbklkIjoxNjIzNzQ3Mjc2NzcxLCJsYXN0RXZlbnRUaW1lIjoxNjIzNzUxMzQyMDc4LCJldmVudElkIjoxMzM4LCJpZGVudGlmeUlkIjo5MTUsInNlcXVlbmNlTnVtYmVyIjoyMjUzfQ==
--- Headers - end ---
```

## Header 편의 조회

```java

@WebServlet(name = "requestHeaderServlet", urlPatterns = "/request-header")
public class RequestHeaderServlet extends HttpServlet {

    // public 말고 protected된 메서드를 사용해야 한다.
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        printHeaderUtils(request);
    }

    // Header 편리한 조회
    private void printHeaderUtils(HttpServletRequest request) {
        System.out.println("--- Header 편의 조회 start ---");

        // Host 헤더
        System.out.println("[Host 편의 조회]");
        System.out.println("request.getServerName() = " +
                request.getServerName());

        // Host 헤더
        System.out.println("request.getServerPort() = " + request.getServerPort());
        System.out.println();

        System.out.println("[Accept-Language 편의 조회]");
        request.getLocales().asIterator()
                .forEachRemaining(locale -> System.out.println("locale = " + locale));

        System.out.println("request.getLocale() = " + request.getLocale());

        System.out.println();
        System.out.println("[Cookie 편의 조회]");

        if (request.getCookies() != null) {
            for (Cookie cookie : request.getCookies()) {
                System.out.println(cookie.getName() + ": " + cookie.getValue());
            }
        }

        System.out.println();
        System.out.println("[Content 편의 조회]");

        System.out.println("request.getContentType() = " + request.getContentType());
        System.out.println("request.getContentLength() = " + request.getContentLength());
        System.out.println("request.getCharacterEncoding() = " + request.getCharacterEncoding());
        System.out.println("--- Header 편의 조회 end ---\n");
    }
}

```

```text
--- Header 편의 조회 start ---
[Host 편의 조회]
request.getServerName() = localhost
request.getServerPort() = 8080

[Accept-Language 편의 조회]
locale = ko_KR
locale = ko
locale = en_US
locale = en
request.getLocale() = ko_KR

[Cookie 편의 조회]
optimizelyEndUserId: oeu1610084455381r0.7319812782071056
ch-veil-id: 9f3b199b-8db0-4d26-91b6-37ba33091c86
lang: ko
_ga: GA1.1.1954188695.1620027883
Idea-d9466cd9: 1bac0d08-b7da-49a6-98bb-ce36acec26a6
ajs_anonymous_id: %22b7ecbc45-f07a-47f9-b0d4-647e30513dde%22
wcs_bt: s_4288e28adaa4:1623751338
amplitude_id_e3025c63ec54f7c9aa28aa2f31de7f2c: eyJkZXZpY2VJZCI6IjM0ZjUwZmVlLWFiZGQtNDgxZC1iMDZiLWE1NDBlOTlkYzhlYVIiLCJ1c2VySWQiOiJTcGpBWVp6eEhIZGJObUVlUGpDdFhWaUprcmYyIiwib3B0T3V0IjpmYWxzZSwic2Vzc2lvbklkIjoxNjIzNzQ3Mjc2NzcxLCJsYXN0RXZlbnRUaW1lIjoxNjIzNzUxMzQyMDc4LCJldmVudElkIjoxMzM4LCJpZGVudGlmeUlkIjo5MTUsInNlcXVlbmNlTnVtYmVyIjoyMjUzfQ==

[Content 편의 조회]
// get 방식은 content가 없기 때문에 null로 나온다.
// post로 body에 데이터를 담아 보내면 text/plain 등이 출력된다.
request.getContentType() = null
request.getContentLength() = -1
request.getCharacterEncoding() = UTF-8
--- Header 편의 조회 end ---
```

## 기타

```java
@WebServlet(name = "requestHeaderServlet", urlPatterns = "/request-header")
public class RequestHeaderServlet extends HttpServlet {

    // public 말고 protected된 메서드를 사용해야 한다.
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        printStartLine(request);
        printHeaders(request);
        printHeaderUtils(request);
        printEtc(request);
    }

    // 기타 정보
    private void printEtc(HttpServletRequest request) { 
        System.out.println("--- 기타 조회 start ---");
        System.out.println("[Remote 정보]");

        System.out.println("request.getRemoteHost() = " + request.getRemoteHost());
        System.out.println("request.getRemoteAddr() = " + request.getRemoteAddr());
        System.out.println("request.getRemotePort() = " + request.getRemotePort());

        System.out.println();
        System.out.println("[Local 정보]");

        System.out.println("request.getLocalName() = " + request.getLocalName());
        System.out.println("request.getLocalAddr() = " + request.getLocalAddr());
        System.out.println("request.getLocalPort() = " + request.getLocalPort());
        System.out.println("--- 기타 조회 end ---\n");
    }
}
```

```text
--- 기타 조회 start ---
[Remote 정보]
request.getRemoteHost() = 0:0:0:0:0:0:0:1
request.getRemoteAddr() = 0:0:0:0:0:0:0:1
request.getRemotePort() = 53043

[Local 정보]
request.getLocalName() = localhost
request.getLocalAddr() = 0:0:0:0:0:0:0:1
request.getLocalPort() = 8080
--- 기타 조회 end ---
```