# 스프링 IoC 컨테이너와 빈

IoC(Inversion Of Control)은 의존 관계 주입(Dependency Injection)이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만드는 게 아니라, 주입 받아 사용하는 방법이다.

```java
public class BookServiceTest {
    @Test
    public void save() {
        BookRepository bookRepository = new BookRepository();
        BookService bookService = new BookService(bookRepository);
    }
}
```

위처럼 IoC는 스프링이 없어도 생성자 등 장치만 있으면 얼마든지 만들 수 있다. 

### BeanFactory

- 스프링 IoC 컨테이너 가장 최상위에 있는 인터페이스
- IoC의 핵심적인 클래스
- 스프링 IoC 컨테이너가 관리하는 객체
- 애플리케이션 컴포넌트의 중앙 저장소
- 빈 설정 소스로부터 빈 정의를 읽어들이고 빈을 구성, 제공함

**Reference**

[Bean Factory](https://docs.spring.io/spring-framework/docs/5.0.8.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)

### Bean

- 스프링 IoC 컨테이너가 관리하는 객체
- 의존성 주입을 하려면 둘 다 빈으로 등록되어야 함

#### 장점

- 싱글톤 스코프의 장점을 사용할 수 있음
    - 싱글톤: 하나를 공유
        - 런타임 시 객체를 매번 만들면 비용이 비싸므로 싱글턴을 이용해 효율적으로 관리함
    - 프로토타입: 매번 다른 객체
- 라이플 사이클 인터페이스
    - 빈이 만들어질 때 등 원하는 시점에 추가 작업을 넣을 수 있음

### ApplicationContext

실질적으로 우리가 많이 사용하게 될 빈 팩토리다. `BeanFactory`를 상속받으므로 `BeanFactory`의 IoC 기능을 가지고 있으면서도 추가적으로 다양한 기능을 가지고 있다.

- 빈 팩토리
- 메시지 소스 처리 기능
- 이벤트 발행 기능
- 리소스 로딩 기능
- etc...

**Reference**

[Application Context](https://docs.spring.io/spring-framework/docs/5.0.8.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)

```java
@Repository
public class BookRepository {
    public Book save(Book book) {
        return null;
    }
}

@Service
public class BookService {
    private BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
    public Book save(Book book) {
        book.setCreated(new Date());
        book.setBookStatus(BookStatus.DRAFT);

        // 사용하는 bookRepository는 null을 리턴하므로 여기도 null을 리턴한다.
        return bookRepository.save(book);
    }
}
```

위 코드는 `BookRepository`에 따라 `BookService`의 리턴값이 달라진다.

```java
public class BookServiceTest {
    @Test
    public void save() {
        Book book = new Book();
        BookService bookService = new BookService(bookRepository);

        Book result = bookService.save(book);

        assertThat(book.getCreated()).isNotNull();
        assertThat(book.getBookStatus()).isEqualTo(BookStatus.DRAFT);
        assertThat(result).isNotNull();
    }
}
```

이렇게 의존성이 있으면 위와 같은 단위 테스트를 하기가 어려워진다. 위는 결국 `null`을 반환할 것이다.

```java
public class BookServiceTest {
    // 가짜 객체를 생성한다.
    @Mock
    BookRepository bookRepository;

    @Test
    public void save() {
        Book book = new Book();
        // 가짜 객체에 mocking 한다.
        // bookRepository의 save()를 호출하면 book을 리턴하라고 설정한 것이다.
        when(bookRepository.save(book)).thenReturn(book);

        BookService bookService = new BookService(bookRepository);

        Book result = bookService.save(book);

        assertThat(book.getCreated()).isNotNull();
        assertThat(book.getBookStatus()).isEqualTo(BookStatus.DRAFT);
        assertThat(result).isNotNull();
    }
}
```

그래서 이렇게 가짜 객체에 mocking 해서 사용한다.