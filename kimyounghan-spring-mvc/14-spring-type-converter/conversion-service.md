# ConversionService

```java
import org.springframework.lang.Nullable;

public interface ConversionService {
    boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);

    boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);

    <T> T convert(@Nullable Object source, Class<T> targetType);

    Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

- 타입 컨버터를 하나하나 만드는 것은 번거롭다.
- 스프링은 개별 컨버터를 모아서 ConversionService를 통해 편리하게 사용할 수 있도록 해준다.

```java
public class ConversionServiceTest {

    @Test
    void conversionService() {
        // 만들어 둔 컨버터를 등록한다.
        DefaultConversionService conversionService = new DefaultConversionService();

        conversionService.addConverter(new IntegerToStringConverter());
        conversionService.addConverter(new StringToIntegerConverter());
        conversionService.addConverter(new IpPortToStringConverter());
        conversionService.addConverter(new StringToIpPortConverter());

        Integer result = conversionService.convert("10", Integer.class);
        assertThat(result).isEqualTo(10);

        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
        assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080));

        String ipPortString = conversionService.convert(new IpPort("127.0.0.1", 8080), String.class);
        assertThat(ipPortString).isEqualTo("127.0.0.1:8080");
    }
}
```

![](../../.gitbook/assets/kimyounghan-spring-mvc/14/screenshot%202022-03-26%20오후%205.51.25.png)

ConversionService에 컨버터를 등록하면 변환해준다.

## 등록과 사용의 분리

```java
conversionService.addConverter(new StringToIpPortConverter());
```

- 컨버터를 등록할 때는 어떤 타입 컨버터인지 알아야 한다.

```java
IpPort ipPort=conversionService.convert("127.0.0.1:8080",IpPort.class);
```

- 컨버터를 사용할 때는 타입 컨버터가 어떤 건지 몰라도 된다.
- 타입 컨버터는 컨버전 서비스 내부에 숨어서 제공된다.
- 사용하는 쪽은 컨버전 서비스 인터페이스에만 의존하면 된다.

### 인터페이스 분리 원칙

```text
클라이언트는 사용하지 않는 메서드에 의존하면 안된다.
```

DefaultConversionService는 두 인터페이스를 구현한다.

- ConversionService
    - 컨버터 사용에 초점을 둔다.
- ConverterRegistry
    - 컨버터 등록에 초점을 둔다.

이렇게 인터페이스가 분리되어 있으면 등록하는 쪽과 사용하는 쪽의 관심사를 분리할 수 있다.

## Converter 적용하기

{% tabs %} {% tab title="WebConfig.java" %}

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new IntegerToStringConverter());
        registry.addConverter(new StringToIntegerConverter());
        registry.addConverter(new IpPortToStringConverter());
        registry.addConverter(new StringToIpPortConverter());
    }
}
```

{% endtab %} {% tab title="HelloController.java" %}

```java

@RestController
public class HelloController {

    @GetMapping("/hello-v2")
    public String helloV2(@RequestParam Integer data) {
        System.out.println("data = " + data);
        return "ok";
    }
}
```

{% endtab %} {% endtabs %}

- 스프링은 내부에서 ConversionService를 제공한다.
    - WebMvcConfigurer.addFormatters()에 컨버터를 등록하면 내부에서 사용하는 ConversionService에 컨버터를 추가해준다.

```text
http://localhost:8080/hello-v2?data=10
```

![](../../.gitbook/assets/kimyounghan-spring-mvc/14/screenshot%202022-03-26%20오후%208.11.41.png)

요청을 날려보면 쿼리 스트링이라 문자인데도 Integer로 파라미터를 잘 받는다.

근데 예전에는 우리가 StringToIntegerConverter를 등록하지 않아도 코드가 잘 실행됐었다. 스프링이 이미 수 많은 기본 컨버터를 제공하기 때문이다.

만약 커스텀 컨버터를 추가하면 기본 컨버터보다 높은 우선 순위를 가진다.

{% tabs %} {% tab title="HelloController.java" %}

```java

@RestController
public class HelloController {

    @GetMapping("/ip-port")
    public String ipPort(@RequestParam IpPort ipPort) {
        System.out.println("ipPort IP = " + ipPort.getIp());
        System.out.println("ipPort PORT = " + ipPort.getPort());
        return "ok";
    }
}
```

{% endtab %} {% endtabs %}

```text
http://localhost:8080/ip-port?ipPort=170.0.0.1:8080
```

![](../../.gitbook/assets/kimyounghan-spring-mvc/14/screenshot%202022-03-26%20오후%208.15.48.png)

커스텀 컨버터도 잘 동작하는 것을 확인했다. 컨버터는 @RequestParam 뿐 아니라 @ModelAttribute 등에도 다 적용이 된다.

### 처리 과정

- @RequestParam을 처리하는 RequestParamMethodArgumentResolver에서
- ConversionService를 사용해 타입을 변환한다.
