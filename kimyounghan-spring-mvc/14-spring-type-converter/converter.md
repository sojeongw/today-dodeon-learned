# Converter

```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {
    T convert(S source);
}
```

- 타입 컨버터를 사용하려면 Converter 인터페이스를 구현한다.

{% tabs %} {% tab title="StringToIntegerConverter.java" %}

```java

@Slf4j
public class StringToIntegerConverter implements Converter<String, Integer> {

    @Override
    public Integer convert(String source) {
        log.info("convert source={}", source);
        return Integer.valueOf(source);
    }
}
```

{% endtab %} {% tab title="IntegerToStringConverter.java" %}

```java

@Slf4j
public class IntegerToStringConverter implements Converter<Integer, String> {

    @Override
    public String convert(Integer source) {
        log.info("convert source={}", source);
        return String.valueOf(source);
    }
}
```

{% endtab %} {% tab title="ConverterTest.java" %}

```java
class ConverterTest {

    @Test
    void stringToInteger() {
        StringToIntegerConverter converter = new StringToIntegerConverter();
        Integer result = converter.convert("10");
        assertThat(result).isEqualTo(10);
    }

    @Test
    void integerToString() {
        IntegerToStringConverter converter = new IntegerToStringConverter();
        String result = converter.convert(10);
        assertThat(result).isEqualTo("10");
    }
}
```

{% endtab %} {% endtabs %}

스트링과 숫자 간의 컨버터를 만들었다.

{% tabs %} {% tab title="IpPort.java" %}

```java

@Getter
@EqualsAndHashCode
public class IpPort {

    private String ip;
    private int port;

    public IpPort(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }
}
```

{% endtab %} {% tab title="StringToIpPortConverter.java" %}

```java

@Slf4j
public class StringToIpPortConverter implements Converter<String, IpPort> {

    @Override
    public IpPort convert(String source) {
        log.info("convert source={}", source);

        // 입력이 127.0.0.1:8080 형태로 들어온다고 가정한다.
        String[] split = source.split(":");

        String ip = split[0];
        int port = Integer.parseInt(split[1]);

        return new IpPort(ip, port);
    }
}
```

{% endtab %} {% tab title="IpPortToStringConverter.java" %}

```java

@Slf4j
public class IpPortToStringConverter implements Converter<IpPort, String> {

    @Override
    public String convert(IpPort source) {
        log.info("convert source={}", source);
        return source.getIp() + ":" + source.getPort();
    }
}
```

{% endtab %} {% endtabs %}

- @EqualsAndHashCode
    - equals & hashcode를 재정의한다.
    - 값이 같으면 같다고 취급한다.

```java
class ConverterTest {

    @Test
    void stringToIpPort() {
        StringToIpPortConverter converter = new StringToIpPortConverter();

        String source = "127.0.0.1:8080";
        IpPort result = converter.convert(source);

        assertThat(result).isEqualTo(new IpPort("127.0.0.1", 8080));
    }

    @Test
    void ipPortToString() {
        IpPortToStringConverter converter = new IpPortToStringConverter();

        IpPort source = new IpPort("127.0.0.1", 8080);
        String result = converter.convert(source);

        assertThat(result).isEqualTo("127.0.0.1:8080");
    }
}
```

테스트 코드로 컨버팅 로직을 확인한다. 하지만 이렇게 하나하나 컨버터를 만드는 건 번거롭다.

### 참고

스프링은 용도에 따라 다양한 타입 컨버터를 제공한다.

- Converter
    - 기본 타입 컨버터
- ConverterFactory
    - 전체 클래스 계층 구조가 필요할 때 사용한다.
- GenericConverter
    - 정교한 구현이 필요할 때 사용한다.
    - 대상 필드의 애너테이션 정보를 사용할 수 있다.
- ConditionalGenericConverter
    - 특정 조건이 참인 경우에만 실행한다.

스프링은 문자, 숫자, boolean, enum 등 일반적인 타입에 대한 컨버터를 기본으로 제공한다. 위 컨버터의 구현체를 찾아보자.

[타입 컨버터](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core-convert)