# 클래스의 분리

상속이 아니라 완전히 독립된 클래스로 분리한다. 객체를 한 번만 만들어 놓고 계속 사용할 수 있다.

```java
// 이제 상속을 하지 않으므로 추상 클래스일 필요가 없다.
public class ConnectionMaker {
    public Connection makeConnection() {
        // DB 커넥션 코드
    }
}
```

{% tabs %}
{% tab title="After" %}
```java
public class UserDAO {
    // 따로 만든 클래스를 이용해서
    private ConnectionMaker connectionMaker;

    public UserDao() {
        // 생성자에서 인스턴스를 만들고
        connectionMaker = new ConnectionMaker();
    }

    // 각각의 메소드에서 사용한다.
    public void add(User user){
        Connection c = connectionMaker.makeConnection();
    }

    public User get(String id){
        Connection c = connectionMaker.makeConnection();
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDAO {

    public void add(User user){
        // DB 커넥션
        Connection c = DriverManager()...

        // statement 실행
        PreparedStatement ps = ...
    }

    public User get(String id){
         // DB 커넥션
         Connection c = DriverManager()...

         // statement 실행
         PreparedStatement ps = ...       
    }
}
```
{% endtab %}
{% endtabs %}

### 단점

분리한 클래스에 종속된다. 즉, 분리한 클래스를 변경하려면 사용하고 있는 클래스도 변경해줘야 한다. 예를 들어 D사에 맞추려면 `ConnectionMaker`를 D사 용으로 다시 만들어야 한다. 이 문제는 우리가 구체적으로 어떤 클래스를 사용할지 `알고 있기` 때문에 일어난다.

