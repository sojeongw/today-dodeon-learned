# 클래스 정보 조회

```java
@Service
public class BookService {

    @Autowired
    BookRepository bookRepository;

}
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class BookServiceTest {
    @Autowired BookService bookService;

    @Test
    public void di() {
        Assert.assertNotNull(bookService.bookRepository);
    }
}
```

어떻게 `bookService.bookRepository`는 `null`이 아닌 걸까?

## 클래스 정보 조회

리플렉션의 시작은 `Class<T>`다. 

```java
public class Book {
    private String a;

    private static String B = "BOOK";
    private static final String C = "BOOK";

    public String d = "d";
    protected String e = "e";

    public Book() {     

    }

    public Book(String a, String d, String e) {
        this.a = a;
        this.d = d;
        this.e = e;
    }

    private void f() {
        System.out.println("F");
    }

    private void g() {
        System.out.println("g");
    }

    private int h() {
        return 100;
    }
}

public interface MyInterface {

}

public class MyBook extends Book implements MyInterface{

}
```

```java
public class App {
    public static void main(String[] args) throws ClassNotFoundException {
        // 클래스 타입에 접근하는 방법
        // 1. 해당 타입을 가지고 가져오기
        Class<Book> bookClass1 = Book.class;
        // 2. 이미 있는 인스턴스에서 가져오기
        Book book = new Book();
        Class<? extends Book> bookClass2 = book.getClass();
        // 3. FQCN으로 가져오기 
        Class<?> bookClass3 = Class.forName("me.dodeon.Book");

        // 정보 가져오기
        // 1. public한 필드만 가져오기
        Arrays.stream(bookClass.getFields()).forEach(System.out::println); 
        // 2. 접근 지정자 상관없이 가져오기
        Arrays.stream(bookClass.getDeclaredFields()).forEach(System.out::println);
    }
}
```

이밖에도 상위 클래스, 인터페이스, 애너테이션, 생성자 등 다양한 정보를 불러올 수 있다.

## 애너테이션과 리플렉션

```java
public @interface MyAnnotation {

}

@MyAnnotation
public class Book {

}
```

내가 아무리 애너테이션을 커스텀으로 만들어서 붙여도 `getAnnotations()`를 실행하면 조회되지 않는다. 애너테이션은 기본적으로 주석과 같은 취급을 받기 때문에 정보가 클래스까지는 남지만 바이트코드를 로딩했을 때 메모리 상에 남지는 않는다.

만약 런타임까지도 이 애너테이션을 유지하고 싶다면 아래와 같이 옵션을 줘야 한다.

```java
// 기본값은 RetentionPolicy.CLASS
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {

}
```

**Reference**

[Class](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)