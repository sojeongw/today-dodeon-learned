# MessageSource

애플리케이션 컨텍스트가 상속하고 있는 또 다른 기능. 국제화(i18n) 기능을 제공하는 인터페이스이다.

## properties를 활용한 방법

```java
@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // message_ko_KR.properties
        System.out.println(messageSource.getMessage("greeting", new String[]{"keesun"}, Locale.KOREA));
        // messages.properties
        // OS 기본 언어 설정에 기반하기 때문에 나는 한국어로 출력된다.
        System.out.println(messageSource.getMessage("greeting", new String[]{"keesun"}, Locale.getDefault()));
    }
}
```

```properties
# messages.properties
# {0}은 넘어오는 파라미터를 의미한다.
greeting=Hello {0}
```

```properties
# messages_ko_KR.properties
greeting=안녕, {0}
```

```text
안녕, keesun
안녕, keesun
```

원래 `ResourceBundleMessageSource` 빈을 등록해야 사용할 수 있지만 스프링 부트는 자동으로 실행해준다.

## 직접 정의하는 방법

`ReloadableResourceBundleMessageSource`를 이용해 직접 정의하는 방법도 있다.

```java
@SpringBootApplication
public class MessageSourceApplication {

    public static void main(String[] args) {
        SpringApplication.run(MessageSourceApplication.class, args);
    }

    // message를 직접 정의하는 방법
    @Bean
    // 빈 이름은 항상 messageSource가 되어야 한다.
    public MessageSource messageSource() {
        var messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:/messages");
        // 한글이 깨지므로 인코딩 해준다.
        messageSource.setDefaultEncoding("UTF-8");
        return messageSource;
    }
}
```

```text
안녕, keesun
안녕, keesun
```

## 메시지 리로딩

중간에 메시지를 수정하면 수정한 내용이 출력된다.

```java
@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        while(true) {
            // message_ko_KR.properties
            System.out.println(messageSource.getMessage("greeting", new String[]{"keesun"}, Locale.KOREA));
            // messages.properties
            // OS 기본 언어 설정에 기반하기 때문에 나는 한국어로 출력된다.
            System.out.println(messageSource.getMessage("greeting", new String[]{"keesun"}, Locale.getDefault()));

            // 1초마다 기록한다.
            Thread.sleep(1000L);
        }
    }
}
```

먼저 메시지가 1초마다 기록되게 만든다.

```java
@SpringBootApplication
public class MessageSourceApplication {

    public static void main(String[] args) {
        SpringApplication.run(MessageSourceApplication.class, args);
    }

    // message를 직접 정의하는 방법
    @Bean
    // 빈 이름은 항상 messageSource가 되어야 한다.
    public MessageSource messageSource() {
        var messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:/messages");

        // 한글이 깨지므로 인코딩 해준다.
        messageSource.setDefaultEncoding("UTF-8");

        // 리소스를 캐시하는 시간을 3초로 설정한다.
        messageSource.setCacheSeconds(3);

        return messageSource;
    }
}
```

그리고 3초마다 캐시를 하게 한 뒤

```properties
greeting=안녕, 반갑습니다. {0}
```

프로퍼티를 수정하고 `Build Project(F9)` 하면 수정 내용이 반영된다. 빌드 패스에 있는 파일을 끌어오는 것이기 때문에 빌드는 반드시 해준다.

```text
안녕, keesun
안녕, keesun
안녕, 반갑습니다. keesun
안녕, 반갑습니다. keesun
```

이렇게 실시간으로 찍힌다.