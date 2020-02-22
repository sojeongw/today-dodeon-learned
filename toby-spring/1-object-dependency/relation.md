---
description: >-
  인터페이스를 사용하더라도, 어떤 클래스로 구현할지 명시해야 하기 때문에 여전히 완전하게 분리되지 않는다. UserDAO 안에 또 다른
  관심사항이 존재하는 것이다.
---

# 관계 설정 책임의 분리

## 클라이언트

여기서 클라이언트는 오브젝트를 사용하는 또 다른 오브젝트를 의미한다. UserDAO를 사용하는 클라이언트 오브젝트에 해당 관심사를 분리한다. 

```java
// 클라이언트 클래스를 생성한다.
public class UserDaoTest {
    public static void main(String[] args) {

        // 여기서 구체적인 클래스를 명시해준다.
        // 만약 N사로 바뀌더라도 이 부분만 변경해주면 된다.
        ConnectionMaker connectionMaker = new DConnectonMaker();

        // 생성자에 파라미터로 넘겨준다.
        UserDao dao = new UserDao(connectionMaker);
    }
}
```

클라이언트에서 인터페이스를 구현하는 클래스를 명시하고 파라미터로 UserDAO에 보내면 파라미터는 인터페이스라는 조건만 충족하면 되므로 UserDAO에서 구체적인 클래스를 알 필요가 없어진다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDAO {
    private ConnectionMaker connectionMaker;

    // 생성자로 인터페이스를 받아온다. 인터페이스 타입만 맞추면 되므로
    // 어떤 클래스가 구체적으로 구현되어 있는지는 알 필요가 없다.
    public UserDao(ConnectionMaker connectonMaker) {
        this.connectionMaker = connectonMaker();
    }

    public void add(User user){
        Connection c = connectionMaker.makeConnection();
    }

    public User get(String id){\
        Connection c = connectionMaker.makeConnection();
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDAO {
    private ConnectionMaker connectionMaker;

    public UserDao() {
        connectionMaker = new DConnectionMaker();
    }

    public void add(User user){
        Connection c = connectionMaker.makeConnection();
    }

    public User get(String id){
        Connection c = connectionMaker.makeConnection();
    }
}
```
{% endtab %}
{% endtabs %}



