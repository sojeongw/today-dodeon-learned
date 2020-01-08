# 오브젝트 스코프
`DaoFactory`를 사용할 때와 애노테이션으로 `애플리케이션 컨텍스트`를 이용할 때 결과는 같지만 중요한 차이점이 있다. 

`오브젝트 팩토리`를 사용하면 실행할 때마다 다른 레퍼런스 주소를 가진다. 반면에 `애플리케이션 컨텍스트`는 `getBean()`으로 호출한 오브젝트가 항상 동일한 객체를 가진다. 이를 `싱글톤`이라고 부른다.

## 서버 애플리케이션과 싱글톤

스프링은 자바 엔터프라이즈 기술을 사용하는 서버를 위해 적용되는 기술이다. 대규모의 서버는 초당 수십에서 수백 번씩 요청을 받아 처리할 수 있어야 한다. 

만약 이런 상황에서 요청이 올 때마다 오브젝트를 새로 만들어 쓴다면 부하가 커질 것이다. 이러한 이유 때문에 싱글톤으로 하나의 오브젝트만 만들어두고 공유하면서 동시에 사용한다.

## 싱글톤 패턴의 한계

### private 생성자

싱글톤 패턴은 생성자를 `private`으로 제한한다. 자기 자신만이 오브젝트를 만들도록 제한하는 것이다. 하지만 상속이 불가능해 다형성을 적용할 수 없다. 객체지향 설계가 불가능하다.

### 테스트의 어려움

싱글톤은 테스트가 어렵거나 아예 불가능하기도 하다. 싱글톤은 오브젝트 생성이 제한되어 있으므로 `mock 오브젝트 등으로 대체하거나 다이나믹하게 주입하기 힘들다. 엔터프라이즈 개발의 핵심인 테스트에 지장이 있다는 것은 큰 단점이다.

### 서버 환경에서 싱글톤 보장의 어려움

서버는 클래스 로더 구성과 JVM 분산 설치에 따라 싱글톤 클래스여도 여러 개의 오브젝트가 만들어질 수 있다.

### 전역 상태의 위험성

싱글톤은 클라이언트가 정해져 있지 않고 언제든 이용할 수 있기 때문에 전역 상태로 사용되기 쉽다. 전역 상태로 자유롭게 접근, 수정, 공유하는 것은 객체 지향 프로그래밍에서 권장되지 않는 방법이다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    // 생성된 싱글톤 오브젝트를 저장할 스태틱 필드를 정의한다.
    // 클래스 외부에서 사용하지 못하도록 private으로 지정한다.
    private static UserDao INSTANCE;
    ...
    private UserDao(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }

    // 이 메소드가 최초로 호출되는 시점에만 오브젝트가 만들어진다.
    // 만들어진 오브젝트는 스태틱 필드에 저장된다.
    // 만들어진 후에는 이미 만들어둔 스태틱 필드의 오브젝트를 넘겨준다.
    public static synchronized UserDao getInstance() {
        if(INSTANCE == null) {
            INSTANCE = new UserDao();
        }
        return INSTANCE;
    }   
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class DaoFactory {
    public UserDao userDao() {
        UserDao userDao = new UserDao(connectionMaker());

        return userDao;
    }

    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```
{% endtab %}
{% endtabs %}

`UserDao`에 싱글톤을 위한 코드를 추가하니 지저분한 느낌이 든다. 생성자는 이제 `private`이기 때문에 `DaoFactory`에서 생성자로 넘겨주는 것이 불가능해졌다.

## 싱글톤 레지스트리

앞서 말한 문제 때문에 스프링은 `싱글톤 레지스트리`라는 싱글톤 오브젝트 생성, 관리 기능을 제공한다. `static` 메소드와 `private` 생성자 없이 평범한 자바 클래스를 싱글톤으로 활용하게 해준다.

덕분에 `public` 생성자를 가질 수 있으며 테스트가 가능하고 이전처럼 생성자 파라미터로 오브젝트를 주입할 수도 있다. 객체 지향적인 설계 방식과 원칙, 디자인 패턴을 적용하는 데 아무런 제약이 없다.

따라서 스프링은 `IoC 컨테이너`일 뿐만 아니라 싱글톤 패턴을 대신해 싱글톤을 만들고 관리하는 `싱글톤 레지스트리`다.

## 싱글톤과 오브젝트 상태

멀티 스레드 환경이라면 여러 스레드가 동시에 접근해서 사용할 수 있으므로 상태 관리에 주의해야 한다. 저장할 공간이 하나 뿐이기 때문에 스레드들이 동시에 수정하면 서로 덮어쓰고 잘못된 값을 읽어올 수 있기 때문이다. 따라서 상태 정보를 가지고 있지 않은 `무상태(stateless)` 방식으로 만들어져야 한다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    // 초기에 설정하면 바뀌지 ㅇ낳는 읽기 전용 인스턴스 변수
    private ConnecitonMaker connectionMaker;

    // 매번 새로운 값으로 바뀌는 정보를 담는 인스턴스 변수
    private Conneciton c;
    private User user;

    public User get(String id){
        this.c = connectionMaker.makeConnection();
        ...
        this.user = new User();
        this.user.setId(rs.getString("id"));

        return this.user;
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDAO {
    private ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker connectonMaker) {
        this.connectionMaker = connectonMaker();
    }

    public void add(User user){
        Connection c = connectionMaker.makeConnection();
    }
}

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

로컬 변수로 선언했었던 `Connection`과 `User`를 인스턴스 필드로 선언했다. 멀티스레드 환경에서 이 필드를 사용하면 오브젝트를 공유하면서 앞서 말한 문제가 발생한다. 따라서 개별로 바뀌는 정보는 로컬 변수나 파라미터로 사용해야 한다.

`ConnectionMaker`는 `DaoFactory`에서 `@Bean`으로 등록했으므로 싱글톤으로 관리되지만 수정하지 않는 정보이므로 문제가 없다.

## 스프링 빈의 스코프

스프링이 관리하는 오브젝트인 `빈`이 생성되고 존재하고 적용되는 범위를 `스코프`라고 한다. 기본 `스코프`는 `싱글톤`이며 `프로토타입`, `request`, `session` 스코프 등이 있다.