# 내장 웹 서버

## 직접 톰캣 띄워보기

- 스프링 부트 자체는 서버가 아니다.
    - 자바 코드로 서버를 만들 수 있는 기능을 제공하는 툴일 뿐이다.

```java

@SpringBootApplication
public class Application {

    public static void main(String[] args) throws LifecycleException {
        // 1. 톰캣 객체 생성
        Tomcat tomcat = new Tomcat();

        // 2. 포트 설정 
        tomcat.setPort(8080);

        // 3. 톰캣에 컨텍스트 추가
        Context context = tomcat.addContext("/", "/");

        // 4. 서블릿 생성
        HttpServlet httpServlet = new HttpServlet() {
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                PrintWriter writer = resp.getWriter();
                writer.println("<html><head><title>");
                writer.println("Hey, Tomcat");
                writer.println("</title><head>");
                writer.println("<body><h1>Hello Tomcat</h1></body>");
                writer.println("<html>");
            }
        };

        // 5. 톰캣에 서블릿 추가
        String servletName = "helloServlet";
        tomcat.addServlet("/", servletName, httpServlet);

        // 6. 컨텍스트에 서블릿 매핑
        // hello라는 요청이 오면 해당 서블릿을 보여준다.
        context.addServletMappingDecoded("/hello", servletName);

        // 7. 톰캣 실행 및 대기
        tomcat.start();
        tomcat.getConnector();
        tomcat.start();
        tomcat.getServer().await();
    }
}
```

- 이 모든 과정을 상세하고 유연하게 설정하고 실행해주는 게 스프링 부트의 자동 설정이다.
- ServletWebServerFactoryAutoConfiguration
    - 서블릿 웹 서버 생성
- TomcatServletWebServerFactoryCustomizer
    - 서버 커스터마이징
- DispatcherServletAutoConfiguration
    - 서블릿 생성 및 등록