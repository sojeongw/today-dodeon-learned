# 스프링 IoC 컨테이너와 빈

- IoC(Inversion Of Control)
    - 의존 관계 주입(Dependency Injection)
    - 어떤 객체가 사용하는 의존 객체를 직접 만드는 게 아니라, 주입 받아 사용하는 방법이다.

```java
class BurgerChef {
    private BurgerRecipe burgerRecipe;

    // 의존하는 객체를 밖에서 파라미터로 주입받는다.
    public BurgerChef(BurgerRecipe burgerRecipe) {
        this.burgerRecipe = burgerRecipe;
    }
}

class BurgerRestaurantOwner {
    private BurgerChef burgerChef = new BurgerChef(new HamburgerRecipe());

    public void changeMenu() {
        burgerChef = new BurgerChef(new CheeseBurgerRecipe());
    }
}
```

IoC는 스프링이 없어도 생성자 등 장치만 있으면 얼마든지 만들 수 있다.

## 스프링 IoC 컨테이너

- BeanFactory
- 애플리케이션 컴포넌트의 중앙 저장소
- 빈 설정 소스로부터 빈 정의를 읽어들이고 빈을 구성하고 제공한다.

## BeanFactory

- 스프링 IoC 컨테이너 가장 최상위에 있는 인터페이스
- IoC의 핵심적인 클래스
- 스프링 IoC 컨테이너가 관리하는 객체
- 애플리케이션 컴포넌트의 중앙 저장소
- 빈 설정 소스로부터 빈 정의를 읽어들이고 빈을 구성, 제공함

**Reference**

[Bean Factory](https://docs.spring.io/spring-framework/docs/5.0.8.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)

## Bean

- 스프링 IoC 컨테이너가 관리하는 객체
- 의존성 주입을 하려면 빈으로 만들어야 한다.

### 장점

- 싱글톤 스코프의 장점을 사용할 수 있다.
    - 하나를 공유하는 방식
    - 런타임 시 객체를 매번 만들면 비용이 비싸므로 싱글턴을 이용해 효율적으로 관리한다.
    - 항상 같은 객체이기 때문에 메모리 면에서 효율적이고 런타임시 성능 최적화에 유리하다.
    - 반대로 프로토타입은?
        - 매번 다른 객체로 다루는 방식
- 라이프 사이클 인터페이스를 지원한다.
    - 빈이 만들어질 때 등 원하는 시점에 추가 작업을 넣을 수 있다.

## ApplicationContext

- 실질적으로 우리가 많이 사용하게 될 빈 팩토리
    - `BeanFactory`를 상속받는다. 
    - `BeanFactory`의 IoC 기능을 가지고 있으면서도 추가적으로 다양한 기능을 가지고 있다.
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

        return bookRepository.save(book);
    }
}
```

`BookRepository`를 뭘 주입받냐에 따라 `BookService`의 리턴값이 달라진다.

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

이렇게 직접 bookService에 의존성이 있으면 단위 테스트를 하기가 어려워진다. 아직 save()를 구현하지 않고 null을 반환하기 때문에 정상적인 테스트가 불가하다.

```java
public class BookServiceTest {
    
    // 가짜 객체를 생성한다.
    @Mock
    BookRepository bookRepository;

    @Test
    public void save() {
        Book book = new Book();
        
        // 가짜 객체에 mocking 한다. = bookRepository의 save()를 호출하면 book을 리턴하라고 설정한다.
        when(bookRepository.save(book)).thenReturn(book);

        BookService bookService = new BookService(bookRepository);
        Book result = bookService.save(book);

        assertThat(book.getCreated()).isNotNull();
        assertThat(book.getBookStatus()).isEqualTo(BookStatus.DRAFT);
        assertThat(result).isNotNull();
    }
}
```

따라서 이렇게 가짜 객체에 mocking 해서 사용한다.