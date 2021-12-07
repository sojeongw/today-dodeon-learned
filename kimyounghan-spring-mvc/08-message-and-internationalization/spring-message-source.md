# 스프링 메시지 소스

스프링은 메시지 관리 기능을 기본적으로 제공한다.

## 직접 등록

```java
class Example {

    @Bean
    public MessageSource messageSource() {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasenames("messages", "errors");
        messageSource.setDefaultEncoding("utf-8");
        return messageSource;
    }
}
```

- baseNames
    - 설정 파일의 이름을 넣는다.
        - `messages` 라고 넣으면 `messages.properties` 파일을 읽는다.
        - 국제화를 적용하려면 `message_en.properties`처럼 파일명 마지막에 언어 정보를 주면 된다.
    - 파일은 `/resoures/messages.properties`에 둔다.
    - 여러 파일을 한 번에 지정할 수 있다.
        - 예제는 `messages`, `errors` 두 개를 지정했다.
- defaultEncoding
    - 인코딩 정보를 지정한다.
    - 주로 utf-8을 사용한다.

## 스프링 부트 자동 설정

스프링 부트는 자동으로 MessageSource를 빈으로 등록한다.

{% tabs %} {% tab title="application.properties" %}

```properties
# 스프링 부트 기본값
# spring.messages.basename=messages
# 쉼표로 여러 파일 설정
spring.messages.basename=messages,config.i18n.messages
```

{% endtab %} {% endtabs %}

- MessageSource를 빈으로 등록하지 않고 스프링 부트 설정도 하지 않으면 기본 설정이 적용된다.
    - `messages_en.proeprties`, `messages_ko.properties`, `messages.properties` 파일을 만들면 자동으로 인식한다.

## 메시지 파일 만들기

`/resources` 아래에 파일을 생성한다.

{% tabs %} {% tab title="messages.properties" %}

```properties
hello=안녕
hello.name=안녕 {0}   # 파라미터를 받을 수도 있다.
```

{% endtab %} {% tab title="messages_en.properties" %}

```properties
hello=hello
hello.name=hello {0}
```

{% endtab %} {% endtabs %}

## 스프링 메시지 소스 사용하기

