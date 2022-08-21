# Spring Application

## 기본 로그 레벨

- INFO

## FailureAnalyzer

- 에러 메시지를 더 예쁘게 출력해주는 기능

## 배너

- 바꾸는 법
    - `src/main/resources`에 배너 파일 추가
        - 파일명은 banner.txt/gif/jpg/png
    - 코드
        - Banner 클래스 구현하고 SpringApplication.setBanner()로 설정한다.
- `${spring-boot.version}` 같은 변수를 쓸 수 있다.
- 위치를 바꾸고 싶다면 properties에 `spring.banner.location`을 바꿔준다.
- 배너를 끄고 싶다면 `setBannerMode()`를 off 한다.

## ApplicationEvent

- 스프링이나 스프링 부트에서 제공하는 다양한 기본 이벤트가 있다.
    - 이벤트 발생 시점은 애플리케이션이 떴을 때, 실패했을 때 등 다양하다.

{% tabs %} {% tab title="SampleListener.java" %}

```java

@Component
public class SampleListener implements ApplicationListener<ApplicationStartedEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("started");
    }
}

```

{% endtab %} {% endtabs %}

- 애플리케이션 컨텍스트가 만들어지고 난 시점
    - 리스너를 빈으로 등록하면 알아서 실행해준다.

{% tabs %} {% tab title="SampleListener.java" %}

```java

@Component
public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartingEvent event) {
        System.out.println("starting");

    }
}
```

{% endtab %} {% tab title="Application.java" %}

```java

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        app.addListeners(new SampleListener());
        app.run(args);
    }

}
```

{% endtab %} {% endtabs %}

- 애플리케이션 컨텍스트를 만들기 전
    - 리스너를 빈으로 등록해도 동작하지 않는다.
- main에서 따로 리스너를 등록해줘야 동작한다.

## WebApplicationType

```java

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        app.addListeners(new SampleListener());
        app.setWebApplicationType(WebApplicationType.SERVLET);
        app.run(args);
    }

}

```

- 기본값
    - SERVLET
- webflux 사용 시
    - REACTIVE
- 둘 다 있다면
    - SERVLET이 우선 순위를 가진다.

## ApplicationArguments

```java

@Component
public class Example {

    public SampleListener(ApplicationArguments arguments) {
        System.out.println("foo: " + arguments.containsOption("foo"));
        System.out.println("bar: " + arguments.containsOption("bar"));
    }
}

```

- IDE Configuration에서 `--`로 들어오는 값
- ApplicationArguments를 빈으로 등록해주기 때문에 언제든 가져다 쓰면 된다.

```java

@Component
public class Example implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("args.containsOption(\"foo\") = " + args.containsOption("foo"));
    }
}
```

- 애플리케이션 실행 뒤 뭔가 하고 싶다면
    - ApplicationRunner
        - 추천
    - CommandLindRunner
- Runner가 여러 개라면 @Order를 달아서 조정할 수 있다.