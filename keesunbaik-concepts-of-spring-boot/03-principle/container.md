# 컨테이너와 서버 포트
## 다른 서블릿 컨테이너로 변경하기

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

## 웹 서버 사용하지 않기

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

## 포트 변경하기

```properties
server.port=7070
```

```text
INFO 33498 --- [           main] o.e.jetty.server.AbstractConnector       : Started ServerConnector@3402b4c9{HTTP/1.1, (http/1.1)}{0.0.0.0:7070}
```

## 랜덤 포트 사용하기

```properties
server.port=0
```

```text
o.e.jetty.server.AbstractConnector       : Started ServerConnector@1816e24a{HTTP/1.1, (http/1.1)}{0.0.0.0:51102}
```

## 설정한 포트 사용하기

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