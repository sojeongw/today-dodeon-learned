# Validation 추상화

> org.springframework.validation.Validator

애플리케이션에서 사용하는 객체 검증용 인터페이스

## 특징

- 어떤 계층과도 관계가 없다.
    - 웹 계층 뿐 아니라 서비스, 데이터 등 다양한 곳에서 사용할 수 있다.
- Bean Validation을 지원한다.
    - Bean Validation
        - 자바 표준 스펙 = Java EE 표준 스펙
        - @NotEmpty, @NotNull 등

## 구현

- 인터페이스를 implemnets할 때는 두 메서드를 구현해야한다.
    - supports()
    - validate()

{% tabs %} {% tab title="EventValidator.java" %}

```java
public class EventValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Event.class.equals(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOfWhitespace(...);
    }
}
```

{% endtab %} {% tab title="AppRunner.java" %}

```java

@Component
public class AppRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        Event event = new Event();
        EventValidator eventValidator = new EventValidator();
        Errors errors = new BeanPropertyBindingResult(event, "event");

        // 검증한다.
        eventValidator.validate(event, errors);
    }
}
```

{% endtab %} {% endtabs %}

## 스프링 부트에서의 validator

{% tabs %} {% tab title="AppRunner.java" %}

```java

@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    Validator validator;

    @Override
    public void run(ApplicationArguments args) {
        Event event = new Event();
        event.title("");
        Errors errors = new BeanPropertyBindingResult(event, "event");

        // 검증한다.
        validator.validate(event, errors);
    }
}
```

{% endtab %} {% tab title="Event.java" %}

```java
public class Event {

    @NotEmpty
    String title;
    
    ...
}

```

{% endtab %} {% endtabs %}

- validator를 빈으로 바로 주입받을 수 있다.

**Reference**

[Bean Validation](https://beanvalidation.org)
