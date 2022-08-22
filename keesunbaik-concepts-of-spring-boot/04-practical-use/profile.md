# 프로파일

## @Profile

{% tabs %} {% tab title="BaseConfiguration.java" %}

```java

@Profile("prod")
@Configuration
public class BaseConfiguration {

    @Bean
    public String hello() {
        return "hello";
    }
}
```

{% endtab %} {% tab title="application.properties" %}

```properties
spring.profiles.active=prod
```

{% endtab %} {% endtabs %}

- spring.profiles.active
    - 프로파일 활성화
- spring.profiles.include
    - 프로파일 추가
- application-{profile}.properties
    - 프로파일용 프로퍼티 파일 생성
    - 