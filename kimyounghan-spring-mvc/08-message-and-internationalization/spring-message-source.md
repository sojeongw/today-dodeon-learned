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
# 스프링 부트 기본값. 적지 않아도 아래의 내용으로 동작한다.
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

## 스프링 메시지 소스 사용

![](../../.gitbook/assets/kimyounghan-spring-mvc/08/screenshot%202022-03-09%20오후%202.13.42.png)

```java

@SpringBootTest
public class MessageSourceTest {

    @Autowired
    MessageSource messageSource;

    @Test
    void helloMessage() {
        String result = messageSource.getMessage("hello", null, null);
        assertThat(result).isEqualTo("안녕");
    }
}
```

- code
    - hello
- args
    - null
- locale
    - null
    - locale 정보가 없으면 basename에서 설정한 기본 이름의 메시지 파일을 조회한다.
    - basename이 messages이므로 messages.properties에서 데이터를 조회한다.

### 메시지가 없는 경우

```java

@SpringBootTest
public class MessageSourceTest {

    @Autowired
    MessageSource messageSource;

    @Test
    void notFoundMessageCode() {
        assertThatThrownBy(() -> messageSource.getMessage("no_code", null, null))
                .isInstanceOf(NoSuchMessageException.class);
    }
}

```

- 설정에 없는 메시지는 NoSuchMessageException을 던진다.

### 기본 메시지 지정

```java

@SpringBootTest
public class MessageSourceTest {

    @Autowired
    MessageSource messageSource;

    @Test
    void notFoundMessageCodeDefaultMessage() {
        String result = messageSource.getMessage("no_code", null, "기본 메시지", null);
        assertThat(result).isEqualTo("기본 메시지");
    }
}
```

- 메시지가 없어도 defaultMessage를 지정해주면 그 데이터를 반환한다.

### 매개 변수 사용

```java

@SpringBootTest
public class MessageSourceTest {

    @Autowired
    MessageSource messageSource;

    @Test
    void argumentMessage() {
        String result = messageSource.getMessage("hello.name", new Object[]{"Spring"}, null);
        assertThat(result).isEqualTo("안녕 Spring");
    }
}
```

- 메시지 설정 파일에 넣은 `{0}`을 매개변수로 치환할 수 있다.

### 국제화 파일 선택

```java

@SpringBootTest
public class MessageSourceTest {

    @Autowired
    MessageSource messageSource;

    @Test
    void defaultLang() {
        assertThat(messageSource.getMessage("hello", null, null)).isEqualTo("안녕");
        assertThat(messageSource.getMessage("hello", null, Locale.KOREA)).isEqualTo("안녕");
    }
}
```

- locale 정보가 없으면 messages를 사용한다.
- locale 정보를 지정했지만 message_ko가 없으므로 messages를 사용한다.

```java

@SpringBootTest
public class MessageSourceTest {

    @Autowired
    MessageSource messageSource;

    @Test
    void enLang() {
        assertThat(messageSource.getMessage("hello", null, Locale.ENGLISH)).isEqualTo("hello");
    }
}
```

- locale 정보가 ENGLISH이므로 message_en을 찾아 사용한다.

## 웹 애플리케이션에 메시지 적용

{% tabs %} {% tab title="messages.properties" %}

```properties
label.item=상품
label.item.id=상품 ID
label.item.itemName=상품명
label.item.price=가격
label.item.quantity=수량
page.items=상품 목록
page.item=상품 상세
page.addItem=상품 등록
page.updateItem=상품 수정
```

{% endtab %} {% tab title="addForm.html" %}

```thymeleafexpressions
<label for="itemName" th:text="#{label.item.itemName}">상품명</label>
```

{% endtab %} {% endtabs %}

## 웹 애플리케이션에 국제화 적용

{% tabs %} {% tab title="messages.properties" %}

```properties
label.item=Item
label.item.id=Item ID
label.item.itemName=Item Name
label.item.price=price
label.item.quantity=quantity
page.items=Item List
page.item=Item Detail
page.addItem=Item Add
page.updateItem=Item Update
button.save=Save
button.cancel=Cancel

```

{% endtab %} {% endtabs %}

- 크롬 설정을 영어로 바꾸고 접속하면 적용된다.
    - 설정을 바꾸면 accept-language 헤더가 언어 정보를 담아 요청하고 해당 언어로 화면이 출력된다.
- 스프링은 기본적으로 Locale 정보를 알아야 언어를 선택할 수 있다.
    - 언어 선택 시 기본으로 accept-language 헤더의 값을 사용한다.

### LocaleResolver

스프링의 Locale 선택 방식을 바꾸고 싶다면 LocaleResolver 인터페이스를 사용한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/08/screenshot%202022-03-09%20오후%202.58.54.png)

- 스프링 부트는 기본적으로 AcceptHeaderLocaleResolver로 accept-language를 사용한다.
- 이 설정을 바꾸고 싶다면 LocaleResolver의 구현체를 쿠키나 세션 기반으로 바꾸면 된다.