# Formatter

- 실무에서는 문자와 다른 타입 간의 변환이 자주 일어난다.
    - ex. 화면에 숫자나 날짜를 문자 형태로 출력해야 할 때
    - 특히 날짜와 숫자 표현은 Locale 즉, 나라마다 달라질 수도 있다.

Formatter는 이렇게 특정한 포맷과 문자 간의 변환을 담당하는 컨버터의 특별한 버전이다.

## Converter vs Formatter

- Converter
    - 범용적인 기능
    - 객체와 객체의 변환
- Formatter
    - 문자에 특화된 기능
    - Locale(현지화) 기능
    - Converter의 특별한 버전

```java
public interface Printer<T> {
    String print(T object, Locale locale);
}

public interface Parser<T> {
    T parse(String text, Locale locale) throws ParseException;
}

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```

- print()
    - 객체를 문자로 변환한다.
- parse()
    - 문자를 객체로 변환한다.

{% tabs %} {% tab title=".java" %}

```java

@Slf4j
public class MyNumberFormatter implements Formatter<Number> {

    @Override
    public Number parse(String text, Locale locale) throws ParseException {
        log.info("text={}, locale={}", text, locale);
        NumberFormat format = NumberFormat.getInstance(locale);

        // 문자를 숫자로 변환한다.
        return format.parse(text);
    }

    @Override
    public String print(Number object, Locale locale) {
        log.info("object={}, locale={}", object, locale);

        // 포맷을 적용한다.
        return NumberFormat.getInstance(locale).format(object);
    }
}
```

{% endtab %} {% endtabs %}

- Number
    - Integer, Long 등 숫자의 부모
- NumberFormat
    - 숫자에 쉼표를 적용하거나 Locale 정보를 활용해 나라 별로 다른 숫자 포맷을 만들어준다.

{% tabs %} {% tab title="MyNumberFormatterTest.java" %}

```java
class MyNumberFormatterTest {

    MyNumberFormatter formatter = new MyNumberFormatter();

    @Test
    void parse() throws ParseException {
        Number result = formatter.parse("1,000", Locale.KOREA);
        assertThat(result).isEqualTo(1000L);
    }

    @Test
    void print() {
        String result = formatter.print(1000, Locale.KOREA);
        assertThat(result).isEqualTo("1,000");
    }
}
```

{% endtab %} {% endtabs %}

테스트 해보면 문자를 숫자로, 숫자를 한국 형식에 맞는 문자로 변환한다.

[Formatter](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format)

## Foramtter를 지원하는 ConversionService

- ConversionService에는 Converter만 등록할 수 있고 Formatter는 불가하다.
- Foramtter도 결국 Converter의 한 종류이기 때문에 Formatter를 지원하는 ConversionService를 통해 추가할 수 있다.

### FormattingConversionService

```java
public class FormattingConversionServiceTest {

    @Test
    void formattingConversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();

        // 컨버터 등록
        conversionService.addConverter(new StringToIpPortConverter());
        conversionService.addConverter(new IpPortToStringConverter());

        // 포매터 등록
        conversionService.addFormatter(new MyNumberFormatter());

        // 컨버터 동작 확인
        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
        assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080));

        // 포매터 동작 확인
        assertThat(conversionService.convert(1000, String.class)).isEqualTo("1,000");
        assertThat(conversionService.convert("1,000", Long.class)).isEqualTo(1000L);
    }
}

```

- Formatter를 지원하는 컨버전 서비스
- DefaultFormattingConversionService
    - FormattingConversionService에 통화, 숫자 등 추가적인 기능을 제공한다.

### 상속 관계

- FormattingConversionService는 ConversionService 기능을 상속받는다.
    - 따라서 컨버터와 포매터 모두 등록할 수 있다.
- 사용할 때는 ConversionService.convert()를 사용하면 된다.
- 스프링 부트는 DefaultFormattingConversionService를 상속 받은 WebConversionService를 사용한다.

## Formatter 적용

{% tabs %} {% tab title=".java" %}

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        /*
        포매터보다 컨버터가 우선 순위가 높기 때문에
        포매터와 비슷한 기능을 수행하는 컨버터를 주석 처리 한다.
        registry.addConverter(new IntegerToStringConverter());
        registry.addConverter(new StringToIntegerConverter());
        */
        registry.addConverter(new IpPortToStringConverter());
        registry.addConverter(new StringToIpPortConverter());

        // 포매터를 추가한다.
        registry.addFormatter(new MyNumberFormatter());
    }
}
```

{% endtab %} {% endtabs %}

```text
# before
${number}: 10000
${{number}}: 10000
${ipPort}: hello.typeconverter.type.IpPort@59cb0946
${{ipPort}}: 127.0.0.1:8080
```

```text
# after
${number}: 10000
${{number}}: 10,000
${ipPort}: hello.typeconverter.type.IpPort@59cb0946
${{ipPort}}: 127.0.0.1:8080
```

- WebConfig에 포매터를 추가한다.
- 숫자에 쉼표를 붙이는 포매터가 적용되었다.