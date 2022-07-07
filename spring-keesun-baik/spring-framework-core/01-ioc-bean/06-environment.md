# Environment

## EnvironmentCapable

- 프로파일과 프로퍼티를 다루는 인터페이스
- 애플리케이션 컨텍스트는 빈 팩토리 기능만 하는 것은 아니다.

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory,
        HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    @Nullable
    String getId();

    String getApplicationName();

    String getDisplayName();

    long getStartupDate();

    @Nullable
    ApplicationContext getParent();

    AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```

- 코드를 보면 여러 인터페이스를 상속받고 있다.
- `EnvironmentCapable`는 크게 두 가지를 제공한다.
    - 그 중 하나가 프로파일이다.

## 프로파일

- 빈들의 묶음.
    - 어떤 환경에서는 이러이러한 빈을 쓰겠다.
- 테스트 환경에서는 a라는 빈을 쓰고 배포 환경에서는 b를 쓰는 등 특정 환경에서 사용할 빈을 바꿔줘야 할 때 사용하는 기능이다.

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // EnvironmentCapable 인터페이스에서 가져온 getEnvironment 메서드
        Environment environment = ctx.getEnvironment();
        // 현재 등록된 프로파일을 불러온다.
        System.out.println(Arrays.toString(environment.getActiveProfiles()));
        // 기본으로 등록된 프로파일을 볼러온다.
        System.out.println(Arrays.toString(environment.getDefaultProfiles()));
    }
}
```

```text
[]
[default]
```

- 현재 등록한 프로파일은 없으며 기본으로 `default` 라는 이름의 프로파일이 등록되어 있다.

```java

@Configuration
@Profile("test")
public class TestConfiguration {
    @Bean
    public BookRepository bookRepository() {
        return new TestBookRepository();
    }
}
```

- 이 빈 설정 파일은 `test` 라는 이름의 프로파일 일 때만 사용된다. 
- 따라서 `test` 프로파일을 쓰기 전까지는 이 빈 설정 파일이 적용되지 않는다.

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ApplicationContext ctx;

    // 주입할 수 없다.
    @Autowired
    BookRepository bookRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // EnvironmentCapable 인터페이스에서 가져온 getEnvironment 메서드
        Environment environment = ctx.getEnvironment();
        // 현재 등록된 프로파일을 불러온다.
        System.out.println(Arrays.toString(environment.getActiveProfiles()));
        // 현재 기본으로 등록된 프로파일을 볼러온다.
        System.out.println(Arrays.toString(environment.getDefaultProfiles()));
    }
}
```

- `BookRepository`가 빈으로 등록되지 않았으므로 당연히 주입도 받을 수 없다.

### 설정 방법

IDE의 `Run/Debug Configuration` 메뉴에서 `Active profiles`에 프로파일 이름을 써준 뒤 아래와 같이 작성한다.

#### 클래스에 정의

```java

@Configuration
@Profile("test")
public class TestConfiguration {
    @Bean
    public BookRepository bookRepository() {
        return new TestBookRepository();
    }
}
```

```text
[test]
[default]
```

다시 실행하면 `test` 프로파일이 출력된다. 하지만 이 방법은 번거롭다.

```java

@Repository
@Profile("test")
public class TestBookRepository implements BookRepository {
}
```

자바 설정 파일 없이 빈으로 사용하고 싶은 곳에 `@Profile` 애너테이션과 프로파일 이름을 설정하면 된다.

#### 메서드에 정의

```java

@Configuration
public class TestConfiguration {
    @Bean
    @Profile("test")
    public BookRepository bookRepository() {
        return new TestBookRepository();
    }
}
```

### 프로파일 표현식

파일 활용

```java

@Repository
// prod가 아닌 프로파일 사용
@Profile("!prod")
public class TestBookRepository implements BookRepository {
}
```

!, &, ||를 이용해 특정 프로파일이 아닌 프로파일의 빈을 설정할 수도 있다.

## 프로퍼티

애플리케이션 등록된 키-값 쌍의 프로퍼티에 접근할 수 있는 기능. 우선 순위가 있는 계층형으로 접근한다. OS 환경 변수나 -D 옵션 프로퍼티 등이 해당한다.

### 설정 방법

#### IDE 활용

IDE에서 `Run/Debug Configurations`의 `VM options`에 `-Dapp.name=spring5`라고 설정할 수 있다.

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ApplicationContext ctx;

    @Autowired
    BookRepository bookRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // EnvironmentCapable 인터페이스에서 가져온 getEnvironment 메서드
        Environment environment = ctx.getEnvironment();
        // 설정한 VM 옵션을 출력한다.
        System.out.println(environment.getProperty("app.name"));
    }
}
```

```text
spring5
```

설정한 값이 제대로 출력된다.

### properties 파일 활용

```properties
app.about=spring
```

```java

@SpringBootApplication
// properties 파일 주소 설정
@PropertySource("classpath:/app.properties")
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ApplicationContext ctx;

    @Autowired
    BookRepository bookRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // EnvironmentCapable 인터페이스에서 가져온 getEnvironment 메서드
        Environment environment = ctx.getEnvironment();
        // 설정한 VM 옵션을 출력한다.
        System.out.println(environment.getProperty("app.name"));
        // properties 파일르 설정한 값을 출력한다.
        System.out.println(environment.getProperty("app.about"));
    }
}
```

```text
spring5
spring
```

properties의 값이 출력되었다. 만약 `app.name` 이라는 같은 키로 설정되었다면 우선 순위는 `VM option`이 높다.