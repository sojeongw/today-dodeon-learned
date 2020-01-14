# DI와 테스트

인터페이스 사용 시 절대 구현 클래스를 바꾸지 않을 예정이면 인터페이스를 DI 해야 할까? 그래도 인터페이스를 두고 DI를 적용해야 한다. 이유는 다음과 같다.

### 바뀌지 않는 소프트웨어는 없다

클래스 대신 인터페이스를 사용하고 new 대신 DI를 하는 것은 아주 간단한 작업이다. 당장에는 클래스를 수정할 계획이 없더라도 언젠가 필요한 상황이 닥치면 시간과 비용의 부담이 크다.

### 인터페이스를 이용해 다른 차원의 기능 도입이 가능하다

1장에서 `UserDao`와 `ConnectionMaker` 사이에 자연스럽게 부가적인 기능을 덧붙일 수 있었던 건 인터페이스 덕분이다. 새로운 기능을 넣기 위해 기존 코드를 수정할 일이 없었다. DI 적용하지 않았다면 불가능했을 것이다.

### 테스트에 유용하다

테스트는 자동으로 실행 가능하며 빠르게 동작해야 한다. 이를 위해서는 가능한 작은 단위를 테스트해야 한다. DI는 테스트를 작은 단위로 만들고 독립적으로 실행되게 하는 데 중요한 역할을 한다.

## 테스트를 위한 @DirtiesContext 

만약 테스트용과 운영용을 분리해서 DB를 사용하고 싶다면 어떻게 해야할까? 테스트할 때 운영용 DB를 사용하면 기존 데이터가 큰 오류가 생길 수 있다. 이런 경우 테스트 코드에 의한 DI를 이용하면 된다.

```java
// 애플리케이션 컨텍스트의 구성이나 상태를 변경한다는 걸 테스트 컨텍스트 프레임워크에 알려준다.
@DirtiesContext
public class UserDaoTest {
    @Autowired
    UserDao dao;

    @Before
    public void setUp() {
        ...
        // 테스트에서 사용할 DataSource 오브젝트를 직접 생성한다.
        DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "id", "pw", true);
        // 코드로 수동 DI 해준다.
        dao.setDataSource(dataSource);
    }
    @Test 
    public void andAndGet() throws SQLException {
	    ...
    }
    
    @Test
    public void count() throws SQLException {
        ...
    }

    @Test
    public void getUserFailure() throws SQLException {
        ...
    }
}
```

이 방식은 애플리케이션 컨텍스트에서 xml 파일에 따라 오브젝트를 가져와 의존 관계를 강제로 변경했기 때문에 조심해야 한다. 애플리케이션 컨텍스트는 스프링 테스트 컨텍스트 프레임워크를 적용했다면 딱 한 개만 만들어지므로 구성이나 상태를 테스트에서 변경하지 않는 게 원칙이다. 그런데 위의 코드는 dao의 의존 관계를 강제로 변경하고 있다.

그래서 `@DirtiesContext`라는 애노테이션을 추가한 것이다. 이는 테스트 컨텍스트 프레임워크에게 이 클래스가 애플리케이션 컨텍스트 상태를 변경한다고 알려준다. 이 애노테이션이 붙은 클래스에서는 애플리케이션 컨텍스트 공유를 허용하지 않고 테스트 매소드마다 새로운 컨텍스트를 만들어 사용한다. 다른 테스트에 영향을 줄 수 있기 때문이다.

`@DirtiesContext`는 메소드 레벨에도 붙일 수 있다. 만약 하나의 메소드에서만 컨텔스트 상태를 변경한다면 메소드에 붙여주는 게 좋다. 테스트가 끝나면 해당 컨텍스트는 폐기되고 새로운 컨텍스트가 만들어진다.

## 테스트를 위한 별도의 DI 설정

위처럼 수동으로 DI 하는 방법은 코드가 많아져 번거롭고 애플리케이션 컨텍스트를 매번 새로 만드는 부담이 있다. 이 경우 `DataSource` 클래스가 빈으로 정의된 `테스트 전용 설정 파일`을 만들면 된다.

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource"> 
<property name="driverClass" value="com.mysql.jdbc.Driver"/> 
<property name="url" value="jdbc:mysql://localhost/testdb"/> 
<property name="username" value="spring" />
<property name="password" value="book" />
</bean>
```

`test-applicationContext.xml`이라는 파일을 만들고 DB url을 바꿔준다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
// 이 부분만 테스트용 xml로 바꿔주고 다른 곳은 그대로 둔다.
@ContextConfguration(locations="/test-applicationContext.xml")
public class UserDaoTest {
    @Autowired
    private UserDao dao;

    private User user1;
    private User user2;
    private User user3;

    @Before
    public void setUp() {
        ...
    }
    @Test 
    public void andAndGet() throws SQLException {
	    ...
    }
    
    @Test
    public void count() throws SQLException {
        ...
    }

    @Test
    public void getUserFailure() throws SQLException {
        ...
    }
}
```

번거롭게 수동 DI 하는 `@DirtiesContext`나 다른 코드 수정 없이 DI가 완료되었다. 이제 애플리케이션 컨텍스트도 하나만 생성되어 공유할 수 있게 되었다.

## 컨테이너 없는 DI 테스트

이번에는 아예 스프링 컨테이너를 사용하지 않고 테스트를 해보자. 테스트 코드에서 직접 오브젝트를 만들고 DI를 하는 것이다.

`UserDaoTest`는 사실 `UserDao`가 DB에 정보를 잘 등록하고 가져오는지만 확인하면 된다. `UserDao`가 스프링 컨테이너에서 동작하는지는 관심사가 아니다.

```java
// @RunWith를 없앤다.
public class UserDaoTest {
    // @Autowired를 없앤다.
    private UserDao dao;

    private User user1;
    private User user2;
    private User user3;

    @Before
    public void setUp() {
        ...
        // 오브젝트 생성, 관계 설정 등을 모두 직접 해준다.
        dao = new UserDao();
        DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "id", "pw", true);
        dao.setDataSource(dataSource);
    }
    @Test 
    public void andAndGet() throws SQLException {
	    ...
    }
    
    @Test
    public void count() throws SQLException {
        ...
    }

    @Test
    public void getUserFailure() throws SQLException {
        ...
    }
}
```

테스트를 위해 `DataSource`를 따로 만드는 번거로움은 있지만 애플리케이션 컨텍스트를 사용하지 않아 코드는 더 단순하고 이해하기 쉽다. 그만큼 시간도 절약된다.

하지만 JUnit은 매번 새로운 테스트 오브젝트를 만드므로 `UserDao`가 매번 새로 만ㄷ르어진다는 단점도 있다. 물론 여기서는 가벼운 오브젝트라 큰 문제는 없다.

이 테스트는 스프링 API에 의존하지 않고 자신의 관심사에만 집중해 깔끔하게 만들어졌다. 이는 DI를 적용했기 때문이다. 즉, DI는 객체지향 프로그래밍 스타일이며 컨테이너가 반드시 필요한 것이 아니다.

### 침투적 기술

특정 API나 인터페이스, 클래스를 사용하도록 강제하는 기술을 말한다. 침투적 기술을 사용하면 애플리케이션 코드가 해당 기술에 종속된다.

### 비침투적 기술

애플리케이션 로직을 담은 코드에 아무런 영향을 주지 않고 적용하는 방식이다. 기술에 종속되지 않은 순수한 코드를 유지할 수 있게 된다. 스프링은 비침투적인 기술의 대표적인 예이며 그래서 스프링 컨테이너 없는 DI 테스트도 가능한 것이다.

## DI를 이용한 테스트 방법 선택

그렇다면 어떤 방법으로 DI를 테스트에 이용해야 할까? 모두 장단점이 있지만 우선적으로 스프링 컨테이너 없이 테스트할 수 있는 방법을 우선적으로 고려하자. 그 밖에는 다음의 특징을 고려해 선택한다.
 
### 오브젝트의 생성과 초기화가 단순할 경우

스프링 컨테이너 없이 테스트한다. 필요한 오브젝트의 생성과 초기화가 단순하다면 이 방법을 가장 먼저 고려해야 한다. 테스트 수행 속도가 가장 빠르고 간결하기 때문이다.

### 복잡한 의존 관계가 있을 경우

스프링 설정을 이용한 DI 방식을 활용한다. 애플리케이션 컨텍스트는 테스트 전용 설정 파일을 따로 만들어 사용하는 게 좋다. 개발 환경과 테스트 환경, 운영 환경이 각각 차이가 있기 때문에 다른 설정 파일을 만드는 게 좋다.

### 예외적인 의존관계를 강제해야할 경우

이때는 컨텍스트에서 DI 받은 오브젝트에 다시 테스트 코드로 수동 DI 하는 방식을 사용한다. `@DirtiesContext` 애노테이션을 잊지 말자.