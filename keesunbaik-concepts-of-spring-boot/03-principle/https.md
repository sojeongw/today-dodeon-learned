# HTTPS와 HTTP2

## HTTPS

### 키스토어 생성

```shell
keytool -genkey -alias tomcat -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 4000
```

명령어를 입력한다.

{% tabs %} {% tab title="application.properties" %}

```properties
server.ssl.key-store=keystore.p12
server.ssl.key-store-password=123456
server.ssl.keyStoreType=PKCS12
server.ssl.keyAlias=tomcat
```

{% endtab %} {% tab title="generate-keystore.sh" %}

```shell
keytool-genkey
        -alias tomcat
        -storetype PKCS12
        -keyalg RSA
        -keysize 2048
        -keystore keystore.p12
        -validity 4000
```

{% endtab %} {% endtabs %}

이제 모든 요청은 https를 붙여야 한다.

### https 요청

```text
Bad Request
This combination of host and port requires TLS.
```

http로 요청하면 Bad Request가 발생한다.

![](../../.gitbook/assets/keesunbaik-concepts-of-spring-boot/03/스크린샷%202022-08-21%20오후%202.35.33.png)

https로 요청하면 이런 화면이 뜬다.

- 서버에 요청을 보낼 때 내가 만든 인증서를 보낸다.
    - 그 인증서는 아까 만든 keystore에 있다.
- 브라우저는 그 인증서의 pubkey를 모르는 상태이기 때문에 이 화면이 나온다.
    - 일반적으로는 공식적으로 발급 받아서 그 인증서를 알고 있기 때문에 제대로 뜬다.

{% tabs %} {% tab title="Application.java" %}

```java

@SpringBootApplication
public class Application {

    @Bean
    public ServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
        tomcat.addAdditionalTomcatConnectors(createStandardConnector());
        return tomcat;
    }

    private Connector createStandardConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setPort(8080);
        return connector;
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

{% endtab %} {% tab title="application.properties" %}

```properties
server.port=8443
```

{% endtab %} {% endtabs %}

http와 https 둘 다 받을 수 있게 설정할 수 있다.


## HTTP2

{% tabs %} {% tab title="application.properties" %}

```properties
server.http2.enabled=true
```

{% endtab %} {% endtabs %}

```text
curl -I -k --http2 https://localhost:8443/hello
HTTP/2 200 
content-type: text/plain;charset=UTF-8
content-length: 0
date: Sun, 21 Aug 2022 05:55:31 GMT
```