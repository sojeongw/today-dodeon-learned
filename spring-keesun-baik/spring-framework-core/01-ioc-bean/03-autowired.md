# @Autowired

## 빈이 없거나 하나인 경우

### 생성자

```java

@Service
public class BookService {
    BookRepository bookRepository;

    @Autowired
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

```java
public class BookRepository {
}
```

- 생성자 주입 시 `BookRepository`에 `@Repository`를 붙이지 않으면 에러가 난다.
- 객체를 생성할 때 해당하는 빈을 찾지 못해 실패하기 때문이다.
- 다른 방법에 비교해 인스턴스를 아예 생성할 수 없다.

**Tip**

- 보통 `Repository`면 `@Component`보다는 `@Repository`로 빈 설정 해주는 것이 좋다.
- 그래야 나중에 `Repository`에만 할 수 있는 설정이나 AOP 적용이 수월하다.

### Setter

```java

@Service
public class BookService {
    BookRepository bookRepository;

    @Autowired
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

```java
public class BookRepository {
}
```

- Setter로 주입할 때도 `@Repository` 애너테이션이 없을 때 에러가 난다.
    - Setter라면 적어도 BookService 자체의 빈/인스턴스는 만들 수 있어야 하는 게 아닌가?
    - 맞다. 하지만 `@Autowired`가 있기 때문에 `BookService` 빈을 만들 때 의존성 주입을 시도하게 되므로 실패하는 것이다.

만약 이 의존성이 필수적인 게 아니라면 아래와 같이 설정할 수 있다.

```java

@Service
public class BookService {
    BookRepository bookRepository;

    @Autowired(required = false)
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

그럼 `BookService` 인스턴스만 생성되고 `BookRepository`는 의존성 주입이 안된채로 빈이 등록된다.

### 필드

```java

@Service
public class BookService {
    @Autowired(required = false)
    BookRepository bookRepository;
}
```

필드 주입에도 똑같이 옵션을 지정할 수 있다.

## 빈이 여러 개인 경우

{% tabs %} {% tab title="BookService.java" %}

```java

@Service
public class BookService {
    @Autowired
    BookRepository bookRepository;
}
```

{% endtab %} {% tab title="BookRepository.java" %}

```java
public interface BookRepository {
}
```

{% endtab %} {% tab title="MyBookRepository.java" %}

```java

@Repository
public class MyBookRepository implements BookRepository {
}
```

{% endtab %} {% tab title="KeesunBookRepository.java" %}

```java

@Repository
public class KeesunBookRepository implements BookRepository {
}
```

{% endtab %} {% endtabs %}

- 같은 `BookRepository`를 구현하는 클래스가 여러 개라면 `BookService`는 `BookRepository`를 주입할 수 없다.

### @Primary

```java

@Repository
@Primary
public class KeesunBookRepository implements BookRepository {
}
```

- 여러 개의 빈 중에 이 빈을 먼저 주입해달라고 할 때 사용한다.

```text
class me.whiteship.autowired.KeesunBookRepository
```

-
- `ApplicationRunner`를 활용해 주입된 빈을 출력하면 위와 같이 나오는 걸 확인할 수 있다.

### @Qualifier

- 적용할 빈 이름을 `@Qualifier`에 추가해준다.
- 빈이 등록될 때 이름의 기본값은 클래스 이름의 앞 부분을 소문자로 바꾼 것이다.

```java

@Service
public class BookService {
    @Autowired
    @Qualifier("keesunBookRepository")
    BookRepository bookRepository;

    public void printBookRepository() {
        System.out.println(bookRepository.getClass());
    }
}
```

```java

@Repository
public class KeesunBookRepository implements BookRepository {
}
```

하지만 `@Primary`가 좀 더 type-safe하므로 `@Primary`를 추천한다.

### 해당 타입의 빈 모두 주입

```java

@Repository
public class KeesunBookRepository implements BookRepository {
}
```

```java

@Repository
public class MyBookRepository implements BookRepository {
}
```

```java

@Service
public class BookService {
    @Autowired
    List<BookRepository> bookRepositories;

    public void printBookRepository() {
        this.bookRepositories.forEach(System.out::println);
    }
}
```

List를 이용해 모든 빈을 다 주입 받을 수도 있다.

```text
me.whiteship.autowired.KeesunBookRepository@172ca72b
me.whiteship.autowired.MyBookRepository@5bda80bf
```

확인하면 두 Repository 모두 주입 받았음을 알 수 있다.

### 클래스 이름 지정

빈을 주입할 때는 타입을 먼저 본 뒤 이름을 본다.

```java

@Service
public class BookService {
    @Autowired
    // 이름 지정
    BookRepository myBookRepository;

    public void printBookRepository() {
        System.out.println(myBookRepository.getClass());
    }
}
```

그래서 주입할 때 특정 클래스 이름을 그대로 쓰면 그 클래스가 주입된다. 하지만 이 방법은 추천하지 않는다.

## 동작 원리

`BeanPostProcessor` 라는 빈 라이프 사이클 인터페이스의 구현체에 의해 동작한다. 다음은 표준 빈 라이프 사이클이다.

1. BeanNameAware's setBeanName
2. BeanClassLoaderAware's setBeanClassLoader
3. BeanFactoryAware's setBeanFactory
4. EnvironmentAware's setEnvironment
5. EmbeddedValueResolverAware's setEmbeddedValueResolver
6. ResourceLoaderAware's setResourceLoader (only applicable when running in an application context)
7. ApplicationEventPublisherAware's setApplicationEventPublisher (only applicable when running in an application
   context)
8. MessageSourceAware's setMessageSource (only applicable when running in an application context)
9. ApplicationContextAware's setApplicationContext (only applicable when running in an application context)
10. ServletContextAware's setServletContext (only applicable when running in a web application context)
11. postProcess**Before**Initialization methods of BeanPostProcessors
12. InitializingBean's afterPropertiesSet
13. a custom init-method definition
14. postProcess**After**Initialization methods of BeanPostProcessors

### BeanPostProcessor

- 새로 만든 빈 인스턴스를 수정할 수 있는 라이프 사이클 인터페이스
- 빈의 인스턴스를 만든 후 초기화할 때 `Initialization` 라이프 사이클이 있는데, 이 라이프 사이클 앞/뒤로 부가적인 작업을 할 수 있는 또 다른 라이프 사이클 콜백이다.
- `11. postProcessBeforeInitialization`, `14. postProcessAfterInitialization` 메서드 같은 콜백을 제공한다.

### AutowiredAnnotationBeanPostProcessor

- `BeanPostProcessor`를 상속하며 `@Autowired`, `@Value`, `@Inject` 애너테이션을 처리해준다. 
- 즉, `11. postProcess**Before**Initialization` 단계에서 `@Autowired`가 붙은 의존성을 주입한다.

### PostConstruct

```java

@Service
public class BookService {
    @Autowired
    BookRepository myBookRepository;

    // 이제 Runner는 필요없다.
    @PostConstruct
    public void setup() {
        System.out.println(myBookRepository.getClass());
    }
}
```

- `@PostConstruct` 단계에서는 이미 빈이 주입된 상태다.
- `12. InitializingBean's afterPropertiesSet` 단계에서 작업한다.
- 이제 `Runner` 없이도 정보를 출력할 수 있다. 출력되는 위치만 조금 다르다.

```text
2020-03-16 13:33:18.519  WARN 14600 --- [           main] o.s.boot.StartupInfoLogger               : InetAddress.getLocalHost().getHostName() took 5005 milliseconds to respond. Please verify your network configuration (macOS machines may need to add entries to /etc/hosts).
2020-03-16 13:33:23.524  INFO 14600 --- [           main] me.whiteship.autowired.DemoApplication   : Starting DemoApplication on macbook.local with PID 14600 (/Users/sojeong/dev/IntelliJ-workspace/spring-keesun-baik/spring-framework-core-02-autowired/target/classes started by sojeong in /Users/sojeong/dev/IntelliJ-workspace/spring-keesun-baik)
2020-03-16 13:33:23.525  INFO 14600 --- [           main] me.whiteship.autowired.DemoApplication   : No active profile set, falling back to default profiles: default
2020-03-16 13:33:24.073  INFO 14600 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-03-16 13:33:24.079  INFO 14600 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-03-16 13:33:24.079  INFO 14600 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.31]
2020-03-16 13:33:24.121  INFO 14600 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-03-16 13:33:24.121  INFO 14600 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 566 ms
// @PostConstruct가 출력되는 위치
class me.whiteship.autowired.MyBookRepository
2020-03-16 13:33:24.219  INFO 14600 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-03-16 13:33:24.342  INFO 14600 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-03-16 13:33:24.347  INFO 14600 --- [           main] me.whiteship.autowired.DemoApplication   : Started DemoApplication in 16.071 seconds (JVM running for 21.668)
// Runner가 출력되는 위치
```

- `Runner`는 구동이 다 되었을 때 일을 한다. 
- 반면, `@PostConstruct`는 `12. InitializingBean's afterPropertiesSet`에 해당한다.

## 정리

1. 빈 팩토리가 `BeanPostProcessor` 타입의 빈을 찾는다.
2. 그 중 하나인 `AutowiredAnnotationBeanPostProcessor`를 찾는다.
3. `AutowiredAnnotationBeanPostProcessor`가 일반적인 빈들에게 beanPostProcessor를 적용한다.

```java

@Component
public class BookServiceRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        AutowiredAnnotationBeanPostProcessor bean
                = applicationContext.getBean(AutowiredAnnotationBeanPostProcessor.class);
        System.out.println(bean);
    }
}
```

`Runner`를 이용해 빈이 등록되어 있는지 확인할 수 있다.