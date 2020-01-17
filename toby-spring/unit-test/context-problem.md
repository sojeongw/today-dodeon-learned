# 애플리케이션 컨텍스트의 문제점

테스트 메소드 실행 과정을 설명하면서 매 테스트 마다 테스트 클래스 오브젝트를 새로 생성한다고 했다. 이는 `애플리케이션 컨텍스트`도 매번 새롭게 만들어진다는 말이다.

빈이 많고 복잡해지면 애플리케이션 컨텍스트 생성에 *시간이 많이 걸린다*. 모든 싱글톤 빈 오브젝트를 초기화할 때 어떤 빈은 자체적인 초기화 작업을 진행해서 더 많은 시간을 필요로 하기 때문이다.

또한, 애플리케이션 컨텍스터가 초기화될 때 특정 빈은 독자적으로 *많은 리소스*를 할당하거나 *독립적인 스레드*를 띄우기도 한다.

테스트는 매번 새로운 오브젝트를 사용하는 것이 원칙이지만 이처럼 많은 자원과 시간이 소모된다면 *테스트 전체가 공유*하는 오브젝트를 만들기도 한다. 이때도 테스트는 순서에 상관없이 일관성 있는 결과를 보장해야 한다.

### @BeforeClass 스태틱 메소드

@BeforeClass는 테스트 클래스 전체에 걸쳐 딱 한 번만 실행되는 스태틱 메소드를 지원한다. 이 메소드에서 애플리케이션 컨텍스트를 만들어 스태틱 변수에 저장하면 된다.

### JUnit의 테스트 컨텍스트 프레임워크

`@BeforeClass`보다 편리하게 사용할 수 있는 기능이다. 간단한 애노테이션 설정으로 모든 테스트가 컨텍스트를 공유할 수 있다.

## 스프링 테스트 컨텍스트 프레임워크 적용

{% tabs %}
{% tab title="After" %}
```java
// 스프링 테스트 컨텍스트 프레임워크의 JUnit 확장 기능을 지정한다.
// JUnit이 테스트를 진행하는 중에 사용할 애플리케이션 컨텍스트를 만들고 관리하는 작업을 진행한다.
@RunWith(SpringJUnit4ClassRunner.class)
// 테스트 컨텍스트가 자동으로 만들어줄 애플리케이션 컨텍스트 파일의 위치를 지정한다.
@ContextConfguration(locations="/applicationContext.xml")
public class UserDaoTest {
    // 테스트 오브젝트가 만들어지면 스프링 테스트 컨텍스트에 의해 자동으로 값이 주입된다.
    @Autowired
    private ApplicationContext context;

    private UserDao dao;
    private User user1;
    private User user2;
    private User user3;

    @Before
    public void setUp() {
        // 기존의 애플리케이션 컨텍스트 코드는 삭제한다.
        this.dao = context.getBean("userDao", UserDao.class);

		this.user1 = new User("gyumee", "박성철", "spring1");
		this.user2 = new User("leegw700", "이길원", "spring2");
		this.user3 = new User("bumjin", "박범진", "spring3");
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
{% endtab %}

{% tab title="Before" %}
```java
public class UserDaoTest {
    private UserDao dao;
    private User user1;
    private User user2;
    private User user3;

    @Before
    public void setUp() {
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
        this.dao = context.getBean("userDao", UserDao.class);

		this.user1 = new User("gyumee", "박성철", "spring1");
		this.user2 = new User("leegw700", "이길원", "spring2");
		this.user3 = new User("bumjin", "박범진", "spring3");
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
{% endtab %}
{% endtabs %}

그런데 이상한 점이 있다. 인스턴스 변수 `context`에 초기화해주는 코드가 없는데 테스트가 성공한다. 이는 JUnit 확장 기능이 자동으로 `context`를 연결해주기 때문이다.

컨텍스트가 만들어질 때 어떤 일이 일어나는지 자세히 살펴보자. 우선 `setUp()`에 오브젝트를 출력하는 코드를 추가한다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfguration(locations="/applicationContext.xml")
public class UserDaoTest {
    @Autowired
    private ApplicationContext context;
    private UserDao dao;
    private User user1;
    private User user2;
    private User user3;

    @Before
    public void setUp() {
        this.dao = context.getBean("userDao", UserDao.class);

		this.user1 = new User("gyumee", "박성철", "spring1");
		this.user2 = new User("leegw700", "이길원", "spring2");
		this.user3 = new User("bumjin", "박범진", "spring3");
        
        // 확인용 코드 추가
        System.out.println(this.context);
        System.out.println(this);
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

결과를 확인하면 이렇게 나온다.

```text
org.springframework.context.support.GenericApplicationContext@d3d6f:
springbook.dao.UserDaoTest@115dD6c

org.springframework.context.support.GenericApplicationContext@d3d6f:
springbook.dao.UserDaoTest@116318b

org.springframework.context.support.GenericApplicationContext@d3d6f: 
springbook.dao.UserDaoTest@15eDc2b
```

`this.context`의 주소값은 모두 동일하고 `UserDaoTest` 오브젝트의 주소값은 매번 다르게 출력되는 걸 볼 수 있다. 즉, 같은 컨텍스트를 공유한다.

JUnit 확장 기능은 테스트가 실행되기 전에 딱 한 번 애플리케이션 컨텍스트를 만든다. 그리고 테스트 오브젝트가 생성될 때마다 애플리케이션 컨텍스트를 테스트 오브젝트의 특정 필드에 `주입`하는 것이다. DI와 유사하지만 오브젝트의 관계를 관리하는 DI와는 조금 성격이 다르다.

이렇게 하면 애플리케이션 컨텍스트를 재사용 할 수 있어 테스트 실행 시간이 줄어들게 된다.

## 테스트 컨텍스트 공유

스프링 테스트 컨텍스트 프레임워크는 여러 개의 테스트 클래스끼리도 같은 설정 파일의 애플리케이션 컨텍스트를 공유할 수 있다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfguration(locations="/applicationContext.xml")
public class UserDaoTest {
    ...
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfguration(locations="/applicationContext.xml")
public class GroupDaoTest {
    ...
}
```

애노테이션을 달아주면 두 테스트 클래스의 모든 테스트 메소드가 하나의 애플리케이션 컨텍스트를 공유하게 된다.

테스트 클래스마다 다른 설정 파일을 사용하거나 특정 클래스에서만 다른 설정 파일을 사용할 수도 있다. 이때도 설정 파일의 종류만큼만 컨텍스트를 만들고 같은 파일을 설정한 클래스는 서로 공유한다.

## Autowired

`@Autowired`는 스프링 DI에 사용되는 특별한 애노테이션이며 다음의 순서로 주입한다.

1. `@Autowired`가 붙은 인스턴스 변수를 찾는다.
2. 테스트 컨텍스트 프레임워크가 컨텍스트 내에서 변수 타입과 일치하는 빈을 찾는다.
3. 타입이 일치하는 빈이 있으면 인스턴스 변수에 주입한다.

일반적으로 주입을 하려면 생성자나 수정자 메소드를 사용해왔지만 이 경우에는 메소드가 없어도 가능하다. 

### 타입에 의한 자동 와이어링

별도로 DI 설정을 하지 않아도 타입 정보만 이용해 빈을 자동으로 가져오는 방법이다. 그런데 잠깐, 이전의 테스트에서는 `ApplicationContext` 빈이 없는데 어떻게 자동으로 필드에 주입한 걸까?

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfguration(locations="/applicationContext.xml")
public class UserDaoTest {

    @Autowired
    private ApplicationContext context;
    private UserDao dao;
    private User user1;
    private User user2;
    private User user3;

    ...
}
```

스프링 애플리케이션 컨텍스트는 초기화할 때 *자기 자신도 빈으로 등록*하기 때문에 `ApplicationContext` 타입의 빈이 이미 컨텍스트에 존재하는 셈이다. 따라서 DI도 가능한 것이다.

`@Autowired`를 이용하면 굳이 컨텍스트를 가져와 `getBean()`을 사용하지 않고 `UserDao` 빈을 직접 DI 받을 수도 있다.

{% tabs %}
{% tab title="After" %}
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfguration(locations="/applicationContext.xml")
public class UserDaoTest {
    // 별도의 설정 없이 타입만으로 바로 주입한다.
    @Autowired
    private UserDao dao;

    private User user1;
    private User user2;
    private User user3;

    @Before
    public void setUp() {
        // 기존의 getBean()은 삭제한다.

		this.user1 = new User("gyumee", "박성철", "spring1");
		this.user2 = new User("leegw700", "이길원", "spring2");
		this.user3 = new User("bumjin", "박범진", "spring3");
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
{% endtab %}

{% tab title="Before" %}
```java
public class UserDaoTest {
    @Autowired
    private ApplicationContext context;

    private UserDao dao;
    private User user1;
    private User user2;
    private User user3;

    @Before
    public void setUp() {
        this.dao = context.getBean("userDao", UserDao.class);

		this.user1 = new User("gyumee", "박성철", "spring1");
		this.user2 = new User("leegw700", "이길원", "spring2");
		this.user3 = new User("bumjin", "박범진", "spring3");
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
{% endtab %}
{% endtabs %}

애플리케이션 컨텍스트를 DI 받아서 DL 방식으로 `UserDao`를 가져올 때보다 코드가 더 깔끔해졌다. `@Autowired`를 지정하기만 하면 어떤 빈이든 다 가져올 수 있다.

이전에 `@Autowired`가 실행되는 순서를 설명했었다.

1. `@Autowired`가 붙은 인스턴스 변수를 찾는다.
2. 테스트 컨텍스트 프레임워크가 컨텍스트 내에서 변수 타입과 일치하는 빈을 찾는다.
3. 타입이 일치하는 빈이 있으면 인스턴스 변수에 주입한다.

만약 같은 타입의 빈이 두 개 이상 있다면 `변수 이름`과 같은 빈이 있는지 확인한다. 변수명으로도 찾을 수 없다면 예외가 발생한다.

> 변수 타입 - 변수 이름 - 예외

### 인터페이스의 적용

`@Autowired`가 할당하는 변수는 클래스 타입은 물론이고 인터페이스 타입도 가능하다. 

인터페이스를 사용할 경우 실제 구현한 클래스와 인터페이스 중 어떤 타입으로 선언하는 것이 나을까? 이는 테스트에서 빈을 어떤 용도로 사용하느냐에 따라 다르다.

```java
public class UserDaoTest {
    @Autowired
    DataSource dataSource;  // DataSource 인터페이스
    ...
}
```

단순히 인터페이스의 정의된 메소드를 사용하고 싶다면 인터페이스 타입으로 받는다. 구현 클래스를 변경하더라도 테스트 코드를 수정할 일이 없기 때문이다.

```java
public class UserDaoTest {
    @Autowired
    SimpleDriverDataSource dataSource;  // DataSource 인터페이스를 구현하는 클래스
    ...
}
```

반면에 구현한 오브젝트 자체에 관심이 있는 경우 해당 구현체 타입으로 받는다. 구현 클래스에 들어있는 DB 연결 정보를 보고 싶거나 특정 메소드를 이용해야 할 경우가 해당한다.

테스트에 필요하다면 애플리케이션 클래스와 밀접한 관계를 맺고 있어도 상관 없다. 코드 내부 구조와 설정을 의도적으로 검증할 필요가 있기 때문이다. 하지만 꼭 필요한 것이 아니라면 테스트에서도 가능한 인터페이스를 이용해 애플리케이션 코드와 느슨한 연결을 해두는 게 좋다.