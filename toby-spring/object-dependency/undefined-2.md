# 인터페이스 도입

## 추상화

추상화란 공통점을 추려서 따로 분리하는 것이다. 두 개의 클래스가 서로 긴밀하게 연결되지 않도록 느슨한 연결고리를 만들어준다. 

실제 구현한 클래스가 무엇인지는 `몰라도` 된다. 그저 해당 인터페이스 타입으로 넘겨주기만 하면 된다. 인터페이스로 사용할 수 있는 기능만 알면 되지, 어떻게 구현했는지는 알 필요가 없다.

{% tabs %}
{% tab title="After" %}
```java
// DB 커넥션 부분을 인터페이스로 분리한다.
public interface ConnectionMaker {
    public Connection makeConnection() {
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public abstract class UserDAO {

    public void add(User user){
        Connection c = getConnection();
    }

    public User get(String id){\
         Connection c = getConnection();     
    }

    private abstract Connection getConnection(){
    }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="After" %}
```java
// 해당 인터페이스를 implements 한 다음
public class DConnectionMaker implements ConnectionMaker {

    // 인터페이스에 있는 메소드를 원하는 대로 구현한다.
    public Connection makeConnection {
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class NUserDAO extends UserDAO {
    public Connection getConnection() {
    }
}

public class DUserDAO extends UserDAO {
    public Connection getConnection() {
    }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="After" %}
```java
public class UserDAO {
    // D사인지 N사인지 명시할 필요 없이 인터페이스만 가져온다.
    private ConnectionMaker connectionMaker;

    public UserDao() {
        // 생성자에서 인터페이스를 구현하는 클래스를 선언한다.
        connectionMaker = new DConnectionMaker();
    }

    public void add(User user){
        // 어떤 클래스를 구현하든 인터페이스를 통하는 것이므로 이 코드에는 변화가 없다.
        Connection c = connectionMaker.makeConnection();
    }

    public User get(String id){\
        Connection c = connectionMaker.makeConnection();
```
{% endtab %}

{% tab title="Before" %}
```java
public abstract class UserDAO {

    public void add(User user){
        Connection c = getConnection();
    }

    public User get(String id){\
         Connection c = getConnection();     
    }

    private abstract Connection getConnection(){
    }
}
```
{% endtab %}
{% endtabs %}

하지만 여기서도 문제가 있다. UserDAO의 생성자에서 `new DConnectionMaker()` 를 써줘야 한다. 어떤 클래스를 사용해야 하는지 알고 있어야 하는 것이다. 그럼 구현하려는 클래스가 다를 경우엔 이 부분을 다시 하나하나 수정해야 한다.

