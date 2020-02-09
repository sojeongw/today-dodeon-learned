# 의존 관계

## 의존 관계 주입\(Dependency Injection\)

![](../../.gitbook/assets/20200108164927.png)

위의 UML 다이어그램처럼 A가 B에 의존한다는 것은 B가 변할 때 A에 영향을 미친다는 뜻이다.

```java
public class UserDAO {
    private ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker connectonMaker) {
        this.connectionMaker = connectonMaker();
    }
    ...
}
```

지금까지 작업했던 `UserDao`는 `ConnectionMaker`에 의존한다. 즉, `ConnectionMaker` 인터페이스를 사용하고 있으며 만약 이것이 바뀌면 영향을 받는다.

하지만 `ConnectionMaker` 인터페이스를 구현한 `DConnectionMaker` 클래스는 `UserDao` 코드 속에 드러나있지 않으며 영향을 주지도 않는다. 이렇듯 인터페이스로 의존 관계를 만들어두면 수정이 자유로워진다.

![](../../.gitbook/assets/20200108165347.png)

이처럼 인터페이스로 느슨한 의존 관계를 맺는 경우는 `UserDao`가 런타임 때 어떤 클래스를 실제로 가져다 쓸 지 알 수 없다. 프로그램이 시작되고 오브젝트가 만들어지고 나서야 알게 되는 것이다. 이때 실제 사용하는 오브젝트를 `의존 오브젝트`라고 한다.

정리하자면 의존 관계 주입은 다음의 조건을 만족해야 한다.

* 클래스 모델이나 코드에는 런타임 때 사용하는 의존 관계가 드러나지 않는다.
* 따라서 인터페이스에만 의존하고 있어야 한다.
* 런타임 시점에 생기는 의존 관계는 컨테이너나 팩토리 같은 제 3자가 결정한다.
* 의존 관계는 사용할 오브젝트에 대한 레퍼런스를 제공, 즉 주입해주면서 만들어진다.

![](../../.gitbook/assets/20200108170833.png)

런타임이 되면 의존 관계 주입이 위와 같이 일어난다. 인터페이스를 실제 구현하는 클래스를 사용하게 되는 것이다.

## 의존 관계 검색\(Dependency Lookup\)

`의존 관계 주입`이 내 자신의 코드가 아니라 외부에서 관계를 맺는 것이라면 `의존 관계 검색`은 능동적으로 필요한 의존 오브젝트를 찾는다. `IoC`이므로 어떤 클래스를 이용할지 결정하지는 않지만 요청할 때 스스로 찾는 방법이다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    // 의존 관계 검색. 능동적으로 자신이 사용할 클래스를 찾는다.
    public UserDao() {
        AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext(DaoFactory.class);
        this.connectionMaker = context.getBean('connectionMaker', ConnectionMaker.class);
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    // 의존 관계 주입. 수동적으로 요청 결과를 받는다.
    public UserDao() {
        DaoFactory daoFactory = new DaoFactory();
        this.connectionMaker = daoFactory.connectionMaker();
    }
}
```
{% endtab %}
{% endtabs %}

`의존 관계 검색`은 `의존 관계 주입`의 거의 모든 장점을 가지고 있으며 방법만 조금 다를 뿐이다. 하지만 코드를 보면 `의존 관계 주입`이 훨씬 깔끔하다.

또한, `의존 관계 검색`에서 검색을 수행하는 오브젝트는 `스프링 빈`으로 등록되어 있을 필요가 없다. 사용할 오브젝트만 `스프링 빈`이면 된다.

## 의존 관계 주입 응용

### 기능 구현의 교환

개발 도중 로컬 DB와 상용 DB를 번갈아 연결해야 할 일이 있다고 가정해보자. 만약 의존성 주입을 하지 않는다면 DB 연결 코드를 사용하는 클래스마다 일일이 수정해야 할 것이다.

{% tabs %}
{% tab title="개발용" %}
```java
public class DaoFactory {
    @Bean
    public ConnectionMaker connectionMaker() {
    return new LocalDBConnectionMaker();
    }
}
```
{% endtab %}

{% tab title="운영용" %}
```java
public class DaoFactory {
    @Bean
    public ConnectionMaker connectionMaker() {
    return new ProductDBConnectionMaker();
    }
}
```
{% endtab %}
{% endtabs %}

`의존 관계 주입`을 이용하면 `ConnectionMaker` 코드만 수정해주면 된다.

### 부가 기능 추가

DAO가 DB 연결을 얼마나 많이 하는지 카운팅 하는 코드를 짜보자.

```java
// 연결 횟수를 세는 코드를 다른 클래스로 분리한다.
public class CountingConnectionMaker implements ConnectionMaker {
    int counter = 0;
    private ConnectionMaker realConnectionMaker;

    // 실제 DB 커넥션을 만들어주는 DConnectionMaker의 오브젝트를 넘겨 받는다.
    public CountingConnectionMaker(ConnectionMaker realConnectionMaker) {
        this.realConnectionMaker = realConnectionMaker;
    }

    // DAO가 연결 메소드를 호출할 때마다 counter를 증가시킨다.
    public Connection makeConnection() {
        this.counter++;

        // 변경된 counter가 반영된 DB 커넥션을 DAO에 넘겨준다.
        return realConnectionMaker.makeConnection();
    }

    public int getCounter() {
        return this.counter;
    }
}
```

DB 연결 횟수를 세는 코드는 DAO의 관심 사항이 아니므로 위처럼 분리한다.

![](../../.gitbook/assets/20200108173345.png)

`CountingConnectionMaker` 적용 전에는 런타임이 되면 `DConnectionMaker`에 의존했다.

![](../../.gitbook/assets/20200108173359.png)

이제는 `CountingConnectionMaker`에 의존하면서 counter를 증가시키고 `CountingConnectionMaker`가 다시 `DConnectionMaker`를 호출해서 실제 DB 커넥션을 제공한다.

{% tabs %}
{% tab title="After" %}
```java
@Configuration
public class CountingDaoFactory {
    @Bean
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }

    @Bean
    public ConnectionMaker connectionMaker() {
        return new CountingConnectionMaker(realConnectionMaker());
    }

    @Bean
    public ConnectionMaker realConnectionMaker() {
    return new DConnectionMaker();
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
@Configuration
public class DaoFactory {

    @Bean  
    public UserDao userDao() {
        UserDao userDao = new UserDao(connectionMaker());

        return userDao;
    }

    @Bean
    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```
{% endtab %}
{% endtabs %}

지금까지 구현한 의존 관계 주입을 위해 새로운 설정 정보를 위와 같이 구성한다.

이제 `ConnectionMaker`는 `CountingConnectionMaker`를 실행할 때 `CountingConnectionMaker` 오브젝트를 만든다. 그리고 `CountingConnectionMaker`는 counter를 반영한 뒤 `realConnectionMaker()`가 실제 DB 연결 오브젝트를 생성하게 한다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDaoConnectionCountingTest {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(CountingDaoFactory.class);
        UserDao dao = context.getBean("userDao" , UserDao .class);

        // 의존 관계 검색으로 가져오고 싶은 빈 이름을 넣는다.
        CountingConnectionMaker ccm = context.getBean("connectionMaker", CountingConnectionMaker.class);

        // counter 값을 출력한다.
        System.out.println("Connection counter " + ccm.getCounter());
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDaoTest {
    public static void main(String[] args) {
        AnnotationConfigurationContext context = new AnnotationConfigurationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);
    }
}
```
{% endtab %}
{% endtabs %}

## 메소드를 이용한 의존 관계 주입

지금까지 의존 관계 주입을 생성자로 구현했다면 이번엔 메소드를 이용해보자.

### 수정자 메소드를 이용한 주입

수정자\(setter\) 메소드는 항상 `set`으로 시작하는 메소드다. 파라미터로 전달된 값을 클래스 내부에 있는 인스턴스 변수에 할당하는 것이다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    private ConnectionMaker connectionMaker;

    // 수정자 메소드를 이용한 의존 관계 주입
    public void setConnectionMaker(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDAO {
    private ConnectionMaker connectionMaker;

    // 생성자를 이용한 의존 관계 주입
    public UserDao(ConnectionMaker connectonMaker) {
        this.connectionMaker = connectonMaker();
    }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="After" %}
```java
@Configuration
public class CountingDaoFactory {
    @Bean
    public UserDao userDao() {
        UserDao userDao = new UserDao();   
        // 수정자 메소드 이용
        userDao.setConnectionMaker(connectionMaker());
        return userDao;
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
@Configuration
public class CountingDaoFactory {
    @Bean
    public UserDao userDao() {
        // 생성자 이용
        return new UserDao(connectionMaker());
    }
}
```
{% endtab %}
{% endtabs %}

### 일반 메소드를 이용한 주입

`set`으로 시작하면서 한 개의 파라미터만 가져야 하는 `수정자 메소드`가 싫다면 일반 메소드를 사용할 수도 있다. 하지만 파라미터가 많아지면 실수하기 쉬우므로 주의해야 한다. 한 번에 모든 파라미터를 다 받아야 하는 생성자보다는 낫다.

