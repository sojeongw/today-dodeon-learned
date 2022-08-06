# ApplicationEventPublisher

- 애플리케이션 컨텍스트가 상속받고 있는 인터페이스.
- 이벤트 프로그래밍에 필요한 인터페이스를 제공하며 옵저버 패턴 구현체다.

## ApplicationEvent를 상속하는 방법

```java
// 이벤트는 빈으로 등록되지 않는다. 원하는 데이터를 전달하는 역할이다.
public class MyEvent extends ApplicationEvent {

    private int data;

    public MyEvent(Object source) {
        super(source);
    }

    public MyEvent(Object source, int data) {
        super(source);
        this.data = data;
    }

    public int getData() {
        return data;
    }
}
```

- 이벤트를 정의한다.

```java

@Component
public class AppRunner implements ApplicationRunner {
    // ApplicationContext가 Publisher를 상속하고 있으므로 이것을 써도 되지만 Publisher를 직접 써도 된다.
    @Autowired
    ApplicationEventPublisher applicationEventPublisher;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // publishEvent로 이벤트를 발생시킨다.
        applicationEventPublisher.publishEvent(new MyEvent(this, 100));
    }
}
```

- 애플리케이션 컨텍스트는 publish 즉, 이벤트 발생 기능을 가지고 있다.

```java

@Component
public class MyEventHandler implements ApplicationListener<MyEvent> {

    @Override
    public void onApplicationEvent(MyEvent myEvent) {
        System.out.println("이벤트 받았다. 데이터는 " + myEvent.getData());
    }
}
```

- 이벤트를 받아서 처리하는 일은 이벤트 핸들러가 담당한다.
    - 이벤트 핸들러는 빈으로 등록해야 한다.

```text
이벤트 받았다. 데이터는 100
```

순서를 보면

1. `AppRunner`의 `publishEvent`가 이벤트를 발생시킨다.
2. 등록되어 있는 빈 중에서 `MyEventHandler`가 이벤트를 받는다.
3. `MyEventHandler`가 해당 이벤트를 출력한다.

## 개선된 버전

스프링 4.2 이후부터는 `MyEvent`가 `ApplicationEvent`를 상속하거나 `MyEventHandler`가 `ApplicationListener`를 상속하는 방법을 쓰지 않아도 된다.

```java
package me.whiteship.eventpublisher;

public class MyEvent {
    private int data;

    // ApplicationEvent는 지웠지만 이벤트를 발생시킨 소스를 갖고 있고 싶다면 이렇게 한다.
    private Object source;

    public MyEvent(Object source, int data) {
        this.source = source;
        this.data = data;
    }

    public Object getSource() {
        return source;
    }

    public int getData() {
        return data;
    }
}
```

```java

@Component
public class MyEventHandler {
    // 이벤트를 처리하는 메서드 위에 에너테이션을 추가해준다.
    // 이제 메서드 이름을 마음대로 바꿔도 된다.
    @EventListener
    public void handle(MyEvent myEvent) {
        System.out.println("이벤트 받았다. 데이터는 " + myEvent.getData());
    }
}
```

- 이벤트 핸들러는 상속은 하지 않더라도 꼭 빈으로 등록되어야 한다.
    - 그래야 누가 처리할지 알 수 있기 때문이다.

## 두 개 이상의 이벤트 처리

- 기본적으로는 순차적으로 처리가 된다.
- 순차적이란 말은 뭐가 먼저 실행될지는 모르지만 동시에 하는 게 아니라 하나씩 실행된다는 말이다.

```java

@Component
public class MyEventHandler {

    @EventListener
    public void handle(MyEvent myEvent) {

        // 스레드 확인용
        System.out.println(Thread.currentThread().toString());
        System.out.println("이벤트 받았다. 데이터는 " + myEvent.getData());
    }
}
```

```java

@Component
public class AnotherHandler {

    @EventListener
    public void handle(MyEvent myEvent) {

        // 스레드 확인용
        System.out.println(Thread.currentThread().toString());
        System.out.println("Another " + myEvent.getData());
    }
}
```

```text
Thread[main,5,main]
Another 100
Thread[main,5,main]
이벤트 받았다. 데이터는 100
```

둘 다 main 스레드에서 실행되고 있으며 순서는 랜덤이다.

### 순서 지정

`@Order`를 사용한다.

```java

@Component
public class MyEventHandler {
    @EventListener
    // 실행 순서 지정
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("이벤트 받았다. 데이터는 " + myEvent.getData());
    }
}
```

```java

@Component
public class AnotherHandler {
    @EventListener
    // 값을 더하면 더 나중에 실행된다.
    @Order(Ordered.HIGHEST_PRECEDENCE + 2)
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("Another " + myEvent.getData());
    }
}
```

```text
Thread[main,5,main]
이벤트 받았다. 데이터는 100
Thread[main,5,main]
Another 100
```

순서를 지정한대로 실행되었다.

### 비동기 실행

`@Async`로 비동기를 적용할 수 있다. 순서는 스레드 스케줄링에 따라 다르므로 보장되지 않는다.

{% tabs %} {% tab title="EventPublisherApplication.java" %}

```java

@SpringBootApplication
// Async 설정
@EnableAsync
public class EventPublisherApplication {

    public static void main(String[] args) {
        SpringApplication.run(EventPublisherApplication.class, args);
    }

}
```

{% endtab %} {% tab title="AnotherHandler.java" %}

```java

@Component
public class AnotherHandler {
    @EventListener
    @Async
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("Another " + myEvent.getData());
    }
}
```

{% endtab %} {% tab title="MyEventHandler.java" %}

```java

@Component
public class MyEventHandler {
    @EventListener
    @Async
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("이벤트 받았다. 데이터는 " + myEvent.getData());
    }
}
```

{% endtab %} {% endtabs %}

```text
Thread[task-2,5,main]
이벤트 받았다. 데이터는 100
Thread[task-1,5,main]
Another 100
```

각각 별도의 스레드에서 실행되었다.

## 스프링이 제공하는 기본 이벤트

- ContextRefreshedEvent
    - ApplicationContext를 초기화 했거나 리프래시 했을 때 발생
- ContextStartedEvent
    - ApplicationContext를 start()하여 라이프사이클 빈들이 시작 신호를 받은 시점에 발생
- ContextStoppedEvent
    - ApplicationContext를 stop()하여 라이프사이클 빈들이 정지 신호를 받은 시점에 발생
- ContextClosedEvent
    - ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에 발생
- RequestHandledEvent
    - HTTP 요청을 처리했을 때 발생

```java

@Component
public class MyEventHandler {
    @EventListener
    @Async
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("이벤트 받았다. 데이터는 " + myEvent.getData());
    }

    @EventListener
    @Async
    public void handle(ContextRefreshedEvent event) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("ContextRefreshedEvent");
    }

    @EventListener
    @Async
    public void handle(ContextClosedEvent event) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("ContextClosedEvent");
    }
}
```

```text
2020-03-17 17:16:22.706  INFO 33745 --- [           main] m.w.e.EventPublisherApplication          : Starting EventPublisherApplication on macbook.local with PID 33745 (/Users/sojeong/dev/IntelliJ-workspace/spring-keesun-baik/spring-framework-core-07-event-publisher/target/classes started by sojeong in /Users/sojeong/dev/IntelliJ-workspace/spring-keesun-baik)
2020-03-17 17:16:22.709  INFO 33745 --- [           main] m.w.e.EventPublisherApplication          : No active profile set, falling back to default profiles: default
2020-03-17 17:16:23.372  INFO 33745 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-03-17 17:16:23.379  INFO 33745 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-03-17 17:16:23.379  INFO 33745 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.31]
2020-03-17 17:16:23.428  INFO 33745 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-03-17 17:16:23.428  INFO 33745 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 672 ms
2020-03-17 17:16:23.576  INFO 33745 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'

Thread[task-1,5,main]
ContextRefreshedEvent

2020-03-17 17:16:23.750  INFO 33745 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-03-17 17:16:23.754  INFO 33745 --- [           main] m.w.e.EventPublisherApplication          : Started EventPublisherApplication in 1.281 seconds (JVM running for 7.094)

Thread[task-3,5,main]
이벤트 받았다. 데이터는 100
Thread[task-2,5,main]
Another 100

//// 애플리케이션을 종료했을 경우 ////

Thread[task-4,5,main]
ContextClosedEvent
2020-03-17 17:17:43.798  INFO 33745 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'

Process finished with exit code 130 (interrupted by signal 2: SIGINT)
```