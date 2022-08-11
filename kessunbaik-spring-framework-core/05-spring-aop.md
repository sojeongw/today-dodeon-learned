# 스프링 AOP

## 개념

- OOP를 보완하는 수단으로 흩어진 Aspect를 모듈화 할 수 있는 프로그래밍 기법
- 흩어져 있으면 유지보수가 어렵다.
- 관심사별로 모은 뒤 적용할 곳을 입력해준다.

![](../.gitbook/assets/keesunbaik-spring-framework-core/04/스크린샷%202022-08-14%20오후%208.37.43.png)

## AOP 주요 개념

- Aspect
    - 각 모듈
- Target
    - Aspect가 적용되는 대상
- Advice
    - 해야할 일
- Join point
    - 끼워넣을 지점
    - ex. 메서드 실행 시
- Pointcut
    - 어디에 적용되어야 하는지 지정하는 곳

## AOP 구현체

- Java
    - AspectJ
    - 스프링 AOP

## 적용 방법

- 컴파일
- 로드 타임
    - 클래스 파일을 로딩하는 시점
- 런타임

## 프록시 기반 AOP

- 프록시 기반 AOP 구현체
- 스프링 빈에만 AOP를 적용할 수 있다.
- 스프링 IoC와 연동해 애플리케이션에서 가장 흔한 문제를 해결하는 것이 목적이다.
    - 모든 AOP 기능을 제공하기 위함이 아니다.

### 프록시 패턴

![](../.gitbook/assets/keesunbaik-spring-framework-core/04/스크린샷%202022-08-14%20오후%208.47.47.png)

- 기존 코드 변경 없이 접근을 제어하거나 부가 기능을 추가할 수 있다.

{% tabs %} {% tab title="AppRunner.java" %}

```java

@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        eventService.createEvent();
        eventService.publishEvent();
    }

}
```

{% endtab %} {% tab title="EventService.java" %}

```java
public interface EventService {

    void createEvent();

    void publishEvent();

}
```

{% endtab %} {% tab title="SimpleEventService.java" %}

```java

@Service
public class SimpleEventService implements EventService {

    // 이 코드는 건드리지 않고 시간을 측정하려 한다.
    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Created an event");
    }

    @Override
    public void publishEvent() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Published an event");
    }

}
```

{% endtab %} {% tab title="ProxySimpleEventService.java" %}

```java
// AppRunner는 이 빈을 먼저 불러와 실행한다.
@Primary
@Service
public class ProxySimpleEventService implements EventService {

    @Autowired
    SimpleEventService simpleEventService;

    // 부가 기능은 여기서 적용한다.
    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.createEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.publishEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }
}
```

{% endtab %} {% endtabs %}

### 문제점

- 매번 프록시 클래스를 작성해야 한다.
- 여러 클래스 여러 메서드에 적용하려면 복잡해진다.

## 스프링 AOP

- 이런 문제를 해결하기 위해 등장했다.
- 스프링 IoC 컨테이너가 제공하는 기반 시설과 Dynamic 프록시를 사용해 복잡한 문제를 해결한다.
    - 스프링 IoC는 기존 빈을 대체하는 동적 프록시 빈을 만들어 등록한다.
    - 클라이언트 코드는 변경되지 않는다.

### 동적 프록시

- 동적으로 프록시 객체를 생성하는 방법
- 자바는 인터페이스 기반의 프록시를 생성한다.
- CGlib은 클래스 기반도 지원한다.

## @AOP

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

- 애너테이션 기반의 스프링 @AOP

### Aspect 정의

- @Aspect
- 빈으로 등록해야 하므로 @Component 추가도 필요하다.

```java

@Component
@Aspect
public class PerfAspect {

    @Around("execution(* me.gracenam..*.EventService.*(..))")
    public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
        long begin = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return retVal;
    }

}
```

### Pointcut 정의

- @Pointcut(표현식)
- 주요 표현식
    - execution
    - @annotation
    - bean
- 포인트컷 조합
    - &&
    - ||
    - !

### Advice 정의

- @Before
- @AfterReturning
- @AfterThrowing
- @Around