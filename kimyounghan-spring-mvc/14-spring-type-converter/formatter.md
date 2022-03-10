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