# ApplicationContext와 다양한 빈 설정 방법

## xml 설정 파일

{% tabs %} {% tab title="DemoApplication.java" %}

```java
public class DemoApplication {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        String[] beanDefinitionNames = context.getBeanDefinitionNames();

        // [bookService, bookRepository]
        System.out.println(Arrays.toString(beanDefinitionNames));

        // 출력된 빈 이름으로 불러온다.
        BookService bookService = (BookService) context.getBean("bookService");

        // true
        System.out.println(bookService.bookRepository != null);
    }
}
```

{% endtab %} {% tab title="BookService.java" %}

```java
public class BookService {
    BookRepository bookRepository;

    public void setBookRepository(BookRepositry bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

{% endtab %} {% tab title="BookRepository.java" %}

```java
public class BookRepository {
}
```

{% endtab %} {% tab title="application.xml" %}

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--  고전적인 방법의 빈 설정 파일  -->
    <!--  id 컨벤션은 소문자로 시작하는 카멜 케이스  -->
    <bean id="bookService"
          class="me.whiteship.springapplicationcontext.BookService">
        <!--  아래까지 해줘야 실제 빈이 주입된다.  -->
        <!--  ref는 레퍼런스로 다른 빈을 참조한다는 의미다. 주입받을 빈의 id를 써준다. -->
        <property name="bookRepository" ref="bookRepository"/>
    </bean>

    <bean id="bookRepository"
          class="me.whiteship.springapplicationcontext.BookRepository">
    </bean>

</beans>
```

{% endtab %} {% endtabs %}

이 방법은 일일이 빈으로 등록하는 것이 굉장히 번거롭다는 것이다.

## 컴포넌트 스캔

그래서 등장한 것이 컴포넌트 스캔이다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="me.whiteship.springapplicationcontext"/>

</beans>
```

- 지정한 패키지를 시작으로 빈을 스캔해서 등록하는 것이다.
- 이때 기본적으로 `@Component`를 이용해 빈으로 등록할 수 있다.
- `@Component`를 확장한 애너테이션도 있다.
    - `@Service`, `@Repository` 등
- 하지만 이 과정은 단지 빈으로 등록하는 것일 뿐이다.
    - 의존성 주입은 `@Autowired`나 `@Inject`를 이용해 따로 해주어야 한다.
    - 단, `@Inject`는 다른 의존성을 필요로 하므로 주로 `@Autowired`를 사용해보자.

{% tabs %} {% tab title="DemoApplication.java" %}

```java
public class DemoApplication {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        String[] beanDefinitionNames = context.getBeanDefinitionNames();

        // [bookService, bookRepository]
        System.out.println(Arrays.toString(beanDefinitionNames));

        // 출력된 빈 이름으로 불러온다.
        BookService bookService = (BookService) context.getBean("bookService");

        // true
        System.out.println(bookService.bookRepository != null);
    }
}
```

{% endtab %} {% tab title="BookService.java" %}

```java

@Service
public class BookService {
    @Autowired
    BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

{% endtab %} {% tab title="BookRepository.java" %}

```java

@Repository
public class BookRepository {
}
```

{% endtab %} {% endtabs %}

애너테이션으로 처리하는 방법은 스프링 2.5부터 지원되었다.

## 자바 설정 파일

xml 대신 자바 파일로 설정하는 방법이다.

{% tabs %} {% tab title="DemoApplication.java" %}

```java
public class DemoApplication {
    public static void main(String[] args) {
        // 자바 설정 파일을 이용해 불러온다.
        ApplicationContext context =
                new AnnotationConfigApplicationContext(ApplicationConfig.class);

        String[] beanDefinitionNames = context.getBeanDefinitionNames();
        System.out.println(Arrays.toString(beanDefinitionNames));

        BookService bookService = (BookService) context.getBean("bookService");
        System.out.println(bookService.bookRepository != null);
    }
}
```

{% endtab %} {% tab title="BookService.java" %}

```java
public class BookService {
    BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

{% endtab %}{% tab title="BookRepository.java" %}

```java
public class BookRepository {
}
```

{% endtab %}{% tab title="ApplicationConfig.java" %}

```java
// 빈 설정 파일임을 알려주는 애너테이션
@Configuration
public class ApplicationConfig {
    @Bean
    public BookRepository bookRepository() {
        return new BookRepository();
    }

    @Bean
    public BookService bookService() {
        BookService bookService = new BookService();
        // 메서드를 호출해서 직접 의존성 주입을 해준다.
        bookService.setBookRepository(bookRepository());
        return bookService;
    }

    /*
    혹은 메서드 파라미터로 주입받을 수 있다.
    @Bean
    public BookService bookService(BookRepository bookRepository) {
        BookService bookService = new BookService();
        bookService.setBookRepository(bookRepository);
        return BookService;
    }
    */
}
```

{% endtab %} {% endtabs %}

아래처럼 `bookService()`에서 직접 의존성 주입을 하지 않고 `@Autowired`를 쓰는 방법도 있다.

{% tabs %} {% tab title="DemoApplication.java" %}

```java
public class DemoApplication {
    public static void main(String[] args) {
        // 자바 설정 파일을 이용해 불러온다.
        ApplicationContext context =
                new AnnotationConfigApplicationContext(ApplicationConfig.class);

        String[] beanDefinitionNames = context.getBeanDefinitionNames();
        System.out.println(Arrays.toString(beanDefinitionNames));

        BookService bookService = (BookService) context.getBean("bookService");
        System.out.println(bookService.bookRepository != null);
    }
}
```

{% endtab %} {% tab title="BookService.java" %}

```java
public class BookService {
    @Autowired
    BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

{% endtab %} {% tab title="BookRepository.java" %}

```java
public class BookRepository {
}
```

{% endtab %} {% tab title="ApplicationConfig.java" %}

```java

@Configuration
public class ApplicationConfig {
    @Bean
    public BookRepository bookRepository() {
        return new BookRepository();
    }

    @Bean
    public BookService bookService() {
        return new BookService();
    }
}
```

{% endtab %} {% endtabs %}

- 직접 의존 관계를 엮지 않아도 빈으로만 등록되어 있다면 `@Autowired`를 사용한 의존성 주입이 이루어진다.

## @ComponentScan 애너테이션

컴포넌트 스캔은 전부 알아서 찾아주니까 편했는데 자바 설정 파일을 쓰니까 하나하나 빈으로 등록해야 해서 불편하다. 이때 `@ComponentScan`을 사용할 수 있다.

{% tabs %} {% tab title="ApplicationConfig.java" %}

```java

@Configuration
// basePackages는 일일이 패키지 이름을 써줘야 하므로 오타로 에러가 날 확률이 있다.
// 하지만 basePackageClasses를 쓰면 조금 더 type-safe 하다.
// 써놓은 클래스가 위치한 곳부터 컴포넌트 스캐닝을 하기 때문이다.
@ComponentScan(basePackageClasses = DemoApplication.class)
public class ApplicationConfig {
}
```

{% endtab %} {% tab title="BookService.java" %}

```java
// 컴포넌트 스캐닝을 위해 다시 애너테이션을 붙여준다.
@Service
public class BookService {
    @Autowired
    BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

{% endtab %} {% tab title="BookRepository.java" %}

```java

@Repository
public class BookRepository {
}
```

{% endtab %} {% endtabs %}

이 방법이 현재 트렌드에 가장 가까운 방법이다.

```java

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {

    }
}
```

- `ApplicationContext`를 직접 쓸 일은 없다.
- 위처럼 `@SpringBootApplication`만 써주면 알아서 해준다.
- `@SpringBootApplication`에 들어가보면 이미 `@ComponentScan`이 들어가있다.
    - 즉, 이 자체가 자바 설정 파일이므로 더 이상 `ApplicationConfig.class` 파일은 필요없다.