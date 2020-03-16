# @Component와 컴포넌트 스캔

## @ComponentScan

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    @AliasFor("basePackages")
    String[] value() default {};

    @AliasFor("value")
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    ...
}
```

`@ComponentScan` 클래스에 들어가면 `@AliasFor("basePackages")`가 보인다. `basePackages`라는 값이 스트링으로 설정되어 있기 때문에 type-safe 하지 않다.

그래서 그것의 대안으로 `basePackageClasses()`라는 메서드가 선언되어 있다. 이 메서드에 전달된 클래스 기준으로 컴포넌트 스캔을 시작한다.

```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

`@ComponentScan`은 `@SpringBootApplication`에 있으므로 `@SpringBootApplication` 애너테이션이 붙은 클래스부터 컴포넌트 스캔을 시작한다. 따라서 여기에 속하지 않는 다른 패키지에 아무리 `@Service` 같은 애너테이션을 붙여도 스캔이 되지 않는다.

## 필터

어떤 애너테이션을 스캔할지 또는 하지 않을지 설정한다.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    ...
}
```

`@ComponentScan`을 쓴다고 해서 모든 것을 다 빈으로 등록하는 것은 아니다. 위 코드는 `excludeFilters`를 사용해 `TypeExcludeFilter`와 `AutoConfigurationExcludeFilter` 클래스를 제외하고 있다.

## 종류

- @Component
    - @Repository
    - @Service
    - @Controller
    - @Configuration
    
## 단점

싱글턴 빈은 초기에 다 생성하기 때문에 등록하는 빈이 많은 경우 구동 시간이 오래걸릴 수 있다. 그 대안으로 펑션을 사용한 빈 등록을 할 수 있다.

## 평션을 이용한 빈 등록

리플렉션, 프록시 등의 기법은 성능에 영향을 주는데 이 방법은 이 두 가지 사용하지 않기 때문에 성능 상의 이점이 있다. 여기서 성능은 애플리케이션 구동 시간을 의미한다.

### SpringApplicationBuilder 사용

```java
public static void main(String[] args) {
    new SpringApplicationBuilder()
        .sources(Demospring51Application.class)
        .initializers((ApplicationContextInitializer<GenericApplicationContext>)
            applicationContext -> {
            applicationContext.registerBean(MyBean.class);
            })
        .run(args);
}
```

### ApplicationContextInitializer 사용

```java
@SpringBootApplication
public class DemoApplication {

    // 아래에서 추가한 빈을 이렇게 사용할 수도 있다.
    @Autowired
    MyService myService;

    public static void main(String[] args) {
        // java 10 부터 추가된 var 타입
        var app = new SpringApplication(DemoApplication.class);

        // 만약 중간에 작업을 추가하고 싶다면 아래 내용을 추가한다.
        // 이 Initializer는 내가 원하는 애플리케이션 컨텍스트를 주입받을 수 있다.
        app.addInitializers((ApplicationContextInitializer<GenericApplicationContext>) ctx -> {
            // 이 GenericApplicationContext을 가지고 직접 원하는 빈을 등록한다.
            ctx.registerBean(MyService.class);
            ctx.registerBean(ApplicationRunner.class, () -> args1
                    -> System.out.println("Functional Bean Definition!!"));
        });

        // 애플리케이션 실행 코드
        app.run(args);
    }
}
```

```text
2020-03-16 20:41:42.916  INFO 19515 --- [           main] m.w.componentscan.DemoApplication        : Starting DemoApplication on macbook.local with PID 19515 (/Users/sojeong/dev/IntelliJ-workspace/spring-keesun-baik/spring-framework-core-03-component/target/classes started by sojeong in /Users/sojeong/dev/IntelliJ-workspace/spring-keesun-baik)
2020-03-16 20:41:42.918  INFO 19515 --- [           main] m.w.componentscan.DemoApplication        : No active profile set, falling back to default profiles: default
2020-03-16 20:41:43.428  INFO 19515 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-03-16 20:41:43.434  INFO 19515 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-03-16 20:41:43.435  INFO 19515 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.31]
2020-03-16 20:41:43.471  INFO 19515 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-03-16 20:41:43.472  INFO 19515 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 526 ms
class me.whiteship.componentscan.MyBookRepository
2020-03-16 20:41:43.571  INFO 19515 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-03-16 20:41:43.717  INFO 19515 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-03-16 20:41:43.721  INFO 19515 --- [           main] m.w.componentscan.DemoApplication        : Started DemoApplication in 1.028 seconds (JVM running for 6.788)
Functional Bean Definition!!
```

이렇게 하면 조건문 등 원하는 코드를 사이에 집어넣을 수 있다는 장점이 있다. 하지만 일일이 등록해야 하므로 컴포넌트 스캔을 대체하는 것은 힘들다.

## 동작 원리

실제 스캐닝은 `ConfigurationClassPostProcessor`라는 `BeanFactoryPostProcessor`에 의해 처리된다.

### BeanFactoryPostProcessor

다른 모든 빈을 만들기 이전에 적용한다. 즉, 다른 빈을 등록하기 전에 컴포넌트 스캔을 해서 등록하는 것이다. 여기서 다른 빈이란 앞의 예시처럼 직접 등록했거나 `@Bean`을 붙여서 만든 빈을 말한다.