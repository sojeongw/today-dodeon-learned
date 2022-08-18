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

## 컨테이너와 서버 포트
### 다른 서블릿 컨테이너로 변경하기

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
      <exclusion>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
      </exclusion>
    </exclusions>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
  </dependency>

....
```

- 기존 톰캣을 빼고 jetty로 변경한다.

### 웹 서버 사용하지 않기

```properties
spring.main.web-application-type=none
```

- 모든 웹서버 설정을 무시하고 없앤다.

```text
2022-08-20 07:24:42.956  INFO 32931 --- [           main] me.whiteship.Application                 : Starting Application using Java 11.0.11 on DodeonM1.local with PID 32931 (/Users/Dodeon/study/keesunbaik-concepts-of-spring-boot/target/classes started by Dodeon in /Users/Dodeon/study/keesunbaik-concepts-of-spring-boot)
2022-08-20 07:24:42.962  INFO 32931 --- [           main] me.whiteship.Application                 : No active profile set, falling back to 1 default profile: "default"
2022-08-20 07:24:43.750  INFO 32931 --- [           main] me.whiteship.Application                 : Started Application in 1.359 seconds (JVM running for 2.105)
holoman = Holoman{name='dodeon', howLong=33}

```

- 웹 서버 실행 없이 끝난다.

### 포트 변경하기

```properties
server.port=7070
```

```text
INFO 33498 --- [           main] o.e.jetty.server.AbstractConnector       : Started ServerConnector@3402b4c9{HTTP/1.1, (http/1.1)}{0.0.0.0:7070}
```

### 랜덤 포트 사용하기

```properties
server.port=0
```

```text
o.e.jetty.server.AbstractConnector       : Started ServerConnector@1816e24a{HTTP/1.1, (http/1.1)}{0.0.0.0:51102}
```

### 설정한 포트 사용하기

{% tabs %} {% tab title=".java" %}

```java
// 웹 서버가 초기화 되면 이 이벤트 리스너가 호출된다.
@Component
public class PortListener implements ApplicationListener<ServletWebServerInitializedEvent> {
    @Override
    public void onApplicationEvent(ServletWebServerInitializedEvent event) {
        ServletWebServerApplicationContext applicationContext = event.getApplicationContext();
        int port = applicationContext.getWebServer().getPort();

        System.out.println("port = " + port);
    }
}

```

{% endtab %} {% endtabs %}

```text
port = 52099
```