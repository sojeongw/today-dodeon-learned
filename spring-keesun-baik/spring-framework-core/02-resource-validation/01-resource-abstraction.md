# Resource 추상화

> org.springframework.core.io.Resource

## 특징

- `java.net.URL`을 추상화 한 것
    - `org.springframework.core.io.Resource` 클래스로 감싸서 실제 로우 레벨에 접근하는 기능을 만들었다.
- 스프링 내부에서 많이 사용하는 인터페이스

## 추상화 한 이유

- 이전에는 `classpath` 기준으로 리소를 읽어오는 기능이 없었다.
- `ServletContext`를 기준을 상대 경로를 읽어오는 기능이 없었다.
- 새로운 핸들러를 등록한 뒤 특별한 URL 접미사를 만들어 사용할 수는 있었지만, 구현이 복잡하고 편의성 메서드가 부족했다.

## Resource 인터페이스

- 주요 메서드
    - getInputStream()
    - exists()
        - 리소스가 항상 존재한다는 보장이 없으므로 확인하는 메서드를 정의해두었다.
    - isOpen()
    - getDescription()
    - etc...

### 구현체

- UrlResource
    - `java.net.URL` 참고
    - http, https, ftp, file, jar 프로토콜을 지원한다.
- ClassPathResource
    - 접두어 `classpath:` 를 지원한다.
- FileSystemResource
- ServletContextResource
    - 읽어들이는 리소스 타입이 애플리케이션 컨텍스트와 관련이 있어서 자주 사용한다.
    - 웹 애플리케이션 루트에서 상대 경로로 리소스를 찾는다.
- etc...

**Reference**

[java.net.URL](https://docs.oracle.com/javase/7/docs/api/java/net/URL.html)

### ApplicationContext에 따라 리소스 읽어오기

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // 클래스 패스 기준으로 문자열에 해당하는 리소스를 찾아서 빈 설정 파일로 활용한다.
        var ctx1 = new ClassPathXmlApplicationContext("config.xml");

        // 파일 시스템 기준으로 문자열에 해당하는 리소스를 찾아서 빈 설정 파일로 활용한다.
        var ctx2 = new FileSystemXmlApplicationContext("config.xml");

        // 애플리케이션 루트 기준으로 문자열에 해당하는 리소스를 찾아서 빈 설정 파일로 활용한다.
        var ctx3 = new WebApplicationContext("config.xml");
    }
}
```

- `Resource`의 타입은 location 문자열과 `ApplicationContext` 타입에 따라 결정된다.
- 만약 `FileSystemXmlApplicationContext`로 불러왔다면, `FileSystemResource`로 리졸빙 한다.
    - FileSystemXmlApplicationContext -> FileSystemResource
    - ClassPathXmlApplicationContext -> ClassPathResource
    - WebApplicationContext -> ServletContextResource

### ApplicationContext에 상관없이 리소스 읽어오기

- ApplicationContext의 타입에 상관없이 리소스 타입을 강제하려면 `java.net.URL 접두어(+ classpath:)`중 하나를 사용할 수 있다.
    - 리소스가 어디서 오는지 명시적으로 나타낼 수 있으므로 이 방식을 더 추천한다.
    - classpath:me/whiteship/config.xml -> ClassPathResource
    - file:///some/resource/path/config.xml -> FileSystemResource
        - 파일을 읽고 싶을 땐 항상 `/`를 세 개로 써줘야 한다.

## 예제

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // 리소스 로더의 타입. 웹 애플리케이션 리소스로 나와야 한다.
        System.out.println(resourceLoader.getClass());

        Resource resource = resourceLoader.getResource("classpath:test.txt");
        // 리소스의 타입. 클래스 패스 리소스가 나와야 한다.
        System.out.println(resource.getClass());

        System.out.println(resource.exists());
        System.out.println(resource.getDescription());
        System.out.println(Files.readString(Path.of(resource.getURI())));
    }
}
```

```text
class org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext
class org.springframework.core.io.ClassPathResource

true
class path resource [test.txt]
hello spring
```

## classpath를 생략한 예제

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // 리소스 로더의 타입. 웹 애플리케이션 리소스로 나와야 한다.
        System.out.println(resourceLoader.getClass());

        // classpath를 뺐을 때는 리소스 로더가 웹 애플리케이션 컨텍스트이므로 서블릿으로 나와야 한다.
        Resource resource = resourceLoader.getResource("test.txt");
        System.out.println(resource.getClass());

        // 따라서 이 리소스를 웹 애플리케이션 루트부터 찾게 된다.
        // 하지만 스프링 부트에서 톰캣은 컨텍스트 패스가 기본적으로 지정되어있지 않다.
        // 결국 리소스를 찾을 수 없다.
        System.out.println(resource.exists());
        System.out.println(resource.getDescription());
        System.out.println(Files.readString(Path.of(resource.getURI())));
    }
}
```

```text
class org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext
class org.springframework.web.context.support.ServletContextResource
false
ServletContext resource [/test.txt]

Caused by: java.io.FileNotFoundException: ServletContext resource [/test.txt] cannot be resolved to URL because it does not exist
```

- 예상한대로 리소스의 타입이 `ServletContextResource`가 되었으며 리소스를 찾을 수 없다는 에러가 발생했다.
- 따라서 스프링 부트 사용 시 `classpath`를 꼭 명시해두자. 

**Reference**

[Interface Resource](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/Resource.html)

[java.net.URL](https://docs.oracle.com/javase/7/docs/api/java/net/URL.html)