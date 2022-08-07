# 데이터 바인딩 추상화

- 프로퍼티의 값을 타겟 객체에 설정하는 것
- 입력값은 대부분 문자열인데, 그 값을 객체가 가지고 있는 int, long, Boolean, Date 등 심지어 Event, Book 같은 도메인 타입으로도 변환해서 넣어주는 기능

## DataBinder

- 데이터 바인딩에 사용되는 인터페이스

## PropertyEditor

- 고전적인 방식
- thread-safe 하지 않다.
    - 상태 정보를 저장하고 있어서 싱글톤 빈으로 등록해 쓰면 큰일난다.
- Object와 String 간의 변환만 할 수 있어 사용 범위가 제한적이다.

```java
public class EventPropertyEditor extends PropertyEditorSupport {

    @Override
    public String getAsText() {
        return ((Event) getValue()).getTitle();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
    }
}
```

여러가지 위험한 점들이 많기 때문에 Spring 3.0부터는 다음의 기능을 사용한다.

## Converter

- S 타입에서 T 타입으로 변환할 수 있는 일반적인 변환기
- 상태 정보가 없어 thread-safe 하다.
- ConverterRegistry에 등록해서 사용한다.

```java
public class StringToEventConverter implements Converter<String, Event> {

    @Override
    public Event convert(String source) {
        Event event = new Event();
        event.setId(Integer.parseInt(source));
        return event;
    }
}
```

## Formatter

- PropertyEditor 대체제
- 좀 더 웹에 최적화된 인터페이스
- Object와 String 간의 변환을 담당한다.
- 문자열을 Locale에 따라 다국화하는 기능도 제공한다.
- FormatterRegistry에 등록해서 사용한다.

```java
public class EventFormatter implements Formatter<Event> {
    @Override
    public Event parse(String text, Locale locale) throws ParseException {
        Event event = new Event();
        int id = Integer.parseInt(text);
        event.setId(id);
        return event;
    }

    @Override
    public String print(Event object, Locale locale) {
        return object.getId().toString();
    }
}
```

## ConversionService

- 지금까지 썼던 Formatter 등의 실제 변환 작업은 이 인터페이스를 통한다.
- thread-safe하게 사용할 수 있다.
- 스프링 MVC, 빈(value) 설정, SpEL에서 사용한다.

![](../../.gitbook/assets/keesunbaik-spring-framework-core/03/스크린샷%202022-08-14%20오후%205.30.35.png)

- DefaultFormattingConversionService
    - ConversionService 역할을 하면서도 FormatterRegistry 역할도 한다.
    - 여러 기본 컨버터와 포매터를 등록해준다.

### 스프링 부트

- 웹 애플리케이션인 경우 스프링 부트가 WebConversionService를 빈으로 등록해준다.
    - DefaultFormattingConversionSerivce를 상속하고 있어 더 많은 기능을 가진다.
- Formatter와 Converter 빈을 찾아 자동으로 등록해 준다.