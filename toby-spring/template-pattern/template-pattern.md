# 템플릿 메소드 패턴

try/catch/finally를 쓰다보니 코드가 많이 복잡해졌다. 이럴 경우 finally에서 하나만 실수로 빼먹어도 커넥션이 반환되지 않고 쌓여 심각한 문제를 일으킬 수 있다.

게다가 예외 상황을 처리하는 코드는 테스트 하기가 매우 어렵고 번거롭다. 적용하더라도 테스트 코드의 양이 방대하게 늘어난다.

아래는 `deleteAll()`에서 변하지 않는 부분만 추려낸 것이다.

```java
public class UserDao {
    ...
    public void deleteAll() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            c = datasource.getConnection();
            ...
            ps.executeUpdate();
        } catch (SQLException e) {  
            throw e;
        } finally {  
            if(ps != null){
                try {
                    ps.close();
                } catch (SQLException e) {  
                }
            }
            if(c != null) {
                try {
                    c.close();
                } catch (SQLException e) {

                }
            }
        }
    }    
}
```

`PreparedStatement`를 만들어서 쿼리를 실행하는 메소드라면 거의 비슷한 구조를 가질 것이다. 이런 코드를 분리해놓으면 재사용이 가능하다.

## 메소드 추출

먼저 메소드로 빼보자. 이 코드는 변하지 않는 부분이 변하는 부분을 감싸고 있어서 변하는 부분만 따로 빼내기가 어렵다. 따라서 변하는 부분을 따로 빼는 방식으로 해본다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    private DataSource dataSource;

    ...

    public void deleteAll() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            c = datasource.getConnection();

            // 새로 만든 메소드를 사용한다.
            ps = makeStatement(c);

            ps.executeUpdate();
        } catch (SQLException e) {  
            throw e;
        } finally {     
            ...
        }
    }    

    private PreparedStatement makeStatement(Connection c) throws SQLExcpetion {
        PreparedStatement ps;
        ps = c.prepareStatement("delete from users");
        return ps;
    }   

    public int getCount() throws SQLException  {
        ...
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    private DataSource dataSource;

    ...

    public void deleteAll() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            c = datasource.getConnection();
            ps = c.prepareStatement("delete from users");
            ps.executeUpdate();
        } catch (SQLException e) {  
            throw e;
        } finally {     
            ...
        }
    }    

    public int getCount() throws SQLException  {
        ...
    }
}
```
{% endtab %}
{% endtabs %}

`makeStatement()`로 분리했지만 DAO 로직마다 새롭게 만들어야 하는 상황이 왔다.

## 템플릿 메소드의 적용

템플릿 메소드 패턴은 상속으로 기능을 확장하는 방법이다. 변하지 않는 부분은 `슈퍼 클래스`에, 변하는 부분은 `추상 메소드`에 정의해서 서브 클래스로 오버라이딩 한 다음 상황에 맞게 수정해서 쓰는 것이다.

{% tabs %}
{% tab title="After" %}
```java
// 추상 메소드를 사용하기 위해 UserDao도 추상 클래스로 변경한다.
abstract public class UserDao {
    ...
    // 자식 클래스에서 구현을 강제하기 위해 추상 메소드로 변경한다.
    abstract protected PreparedStatement makeStatement(Connection c) throws SQLException;
    ...
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    ...

    private PreparedStatement makeStatement(Connection c) throws SQLExcpetion {
        PreparedStatement ps;
        ps = c.prepareStatement("delete from users");
        return ps;
    }  
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="After" %}
```java
// 추상 클래스 UserDao를 상속하는 서브 클래스를 만든다. 
public class UserDaoDeleteAll extends UserDao {
    // 추상 메소드를 재정의한다.
    protected PreparedStatement makeStatement(Connection c) throws SQLException {
        PreparedStatement ps = c.prepareStatement("delete from users");
        return ps;
    }   
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    ...

    private PreparedStatement makeStatement(Connection c) throws SQLExcpetion {
        PreparedStatement ps;
        ps = c.prepareStatement("delete from users");
        return ps;
    }  
}
```
{% endtab %}
{% endtabs %}

상속으로 원하는 만큼 확장할 수 있고 DAO 클래스를 굳이 수정할 필요도 없다. 하지만 DAO에 어떤 로직 만들 때마다 상속으로 새로운 클래스도 만들어야 한다는 부담이 생긴다.

![](../../.gitbook/assets/template-method-pattern.png)

이처럼 `UserDao`의 JDBC 메소드가 4개면 서브 클래스도 4개여야 한다. 또한, 확장하는 방법이 이미 클래스를 만들 때부터 고정되어 버린다. 런타임이 아니라 컨파일 시점에 클래스 간의 관계가 설정되어 있는 것이다. 결국 유연성이 떨어지는 코드가 된다.

## 전략 패턴

개방 폐쇄 원칙\(OCP\)를 지키면서도 템플릿 메소드 패턴보다 유연하게 만드려면 인터페이스를 활용한다. 즉, 변하는 부분만 클래스로 분리하고 인터페이스를 이용해 위임하는 것이다.

![](../../.gitbook/assets/strategy-pattern.png)

`컨텍스트`는 변하지 않는 부분, `전략`은 변하는 부분이다. 일정한 구조를 가진 `컨텍스트`로 동작하다가 특별하게 확장해야 하는 기능은 `Strategy` 인터페이스를 사이에 두고 외부의 독립된 전략 클래스로 넘긴다.

### deleteAll\(\)의 컨텍스트

`deleteAll()`의 변하지 않는 부분은 JDBC를 이용해 DB를 업데이트 하는 작업이다.

* DB 커넥션 가져오기
* PreparedStatement를 만들어줄 외부 기능 호출하기
* 전달받은 PreparedStatement 실행하기
* 예외가 발생하면 다시 메소드 밖으로 던지기
* PreparedStatement와 Connection 닫아주

여기서 `PreparedStatement`를 만들어주는 외부 기능이 `전략`에 해당한다. 이 기능을 인터페이스로 만든 다음 인터페이스의 메소드로 전략을 호출하면 된다.

전략을 호출할 때는 컨텍스트에서 미리 만들어 둔 `Connection`을 같이 보내줘야 `PreparedStatement`를 만들 수 있다.

```java
package springbook.user.dao;
    ...
    public interface StatementStrategy {
        // 컨텍스트가 Connection을 전달하면 PreparedStatement를 만들어서 반환한다.
        PreparedStatement makePreparedStatement(Connection c) throws SQLException {
    }
}
```

{% tabs %}
{% tab title="After" %}
```java
package springbook.user.dao;

// UserDao를 상속하는 대신 인터페이스를 구현하는 전략 클래스를 사용한다.
public class DeleteAllStatement implements StatementStrategy {
    // 인터페이스의 메소드를 상속해서 실제 전략 즉, PreparedStatement 생성을 구현한다.
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        PreparedStatement ps = c.prepareStatement("delete from users");
        return ps;
    } 
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDaoDeleteAll extends UserDao {
    protected PreparedStatement makeStatement(Connection c) throws SQLException {
        PreparedStatement ps = c.prepareStatement("delete from users");
        return ps;
    }   
}
```
{% endtab %}
{% endtabs %}

이제 `PreparedStrategy`를 확장한 전략 클래스 `DeleteAllStatement`가 만들어졌다. 이것을 `contextMethod()`에 해당하는 `UserDao`의 `deleteAll()`에서 사용하면 된다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    private DataSource dataSource;

    ...

    public void deleteAll() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            c = datasource.getConnection();

            // 인터페이스를 실제 구현하는 전략 클래스를 만들고
            StatementStrategy strategy = new DeleteAllStatement();
            // 그 안에서 재정의된 메소드에 커넥션 정보를 보내준다.
            ps = strategy.makePreparedStatement(c);

            // 전략이 적용된 쿼리를 실행한다.
            ps.executeUpdate();
        } catch (SQLException e) {  
            throw e;
        } finally {     
            ...
        }
    }    

    public int getCount() throws SQLException  {
        ...
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    private DataSource dataSource;

    ...

    public void deleteAll() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            c = datasource.getConnection();

            ps = makeStatement(c);
            ps.executeUpdate();
        } catch (SQLException e) {  
            throw e;
        } finally {     
            ...
        }
    }    

    public int getCount() throws SQLException  {
        ...
    }
}
```
{% endtab %}
{% endtabs %}

그런데 이상한 점이 있다. 전략 패턴은 컨텍스트를 그대로 유지하면서 전략만 바꿔 쓴다는 OCP의 원칙을 사용한 것이다. 그런데 이미 컨텍스트에서 `어떤 구현 클래스를 쓸지` 고정되어 버렸다.

### 클라이언트/컨텍스트 분리

컨텍스트가 어떤 전략을 사용할지는 클라이언트가 정한다. 클라이언트가 사용할 전략을 선택해서 컨텍스트에 오브젝트로 전달한다. 그리고 컨텍스트는 전달받은 전략의 구현 클래스를 사용한다.

그림으로 나타내면 아래와 같다.

![](../../.gitbook/assets/client-context.png)

이 구조는 이전에 배웠던 구조와 비슷하다.

![](../../.gitbook/assets/client-context2.png)

컨텍스트인 `UserDao`는 `ConnectionMaker`라는 전략을 필요로 한다. 클라이언트인 `UserDaoTest`는 필요한 전략을 만들어서 보내준다.

![](../../.gitbook/assets/object-factory.png)

그리고 이때 썼던 `ObjectFactory`는 전략 오브젝트를 만들고 컨텍스트로 전달하는 책임을 따로 분리시킨 클래스였다.

이러한 과정을 일반화 한 것이 바로 `의존 관계 주입`이다. DI란 결국 전략 패턴을 일반적으로 활용할 수 있도록 만든 구조이다.

이제 JDBC try/catch/finally 코드를 독립시켜보자.

```java
StatementStrategy strategy = new DeleteAllStatement();
```

이 코드를 제외하면 모두 컨텍스트 코드이므로 별도의 메소드로 분리한다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    ...

    // deleteAll()이 클라이언트를 담당한다. 즉, 전략 오브젝트를 만들고 컨텍스트를 호출한다.
    public void deleteAll() throws SQLException {
        // 사용하려는 전략을 생성한다.
        StatementStrategy st = new DeleteAllStatement();
        // 컨텍스트를 호출하면서 전략을 넘겨준다.
        jdbcContextWithStatementStrategy(st);
    }

    // 이 컨텍스트를 호출할 떄 클라이언트가 StatementStrategy라는 전략을 넘겨준다.
    public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            c = dataSource.getConnection();

            // 전략은 생성이 필요한 시점에 호출해서 사용한다.
            ps = stmt.makePreparedStatement(c);

            ps.executeUpdate();
        } catch (SQLException e) {
            throw e;
        } finally {
            if (ps != null) { try { ps.close(); } catch (SQLException e) {} }
            if (c != null) { try {c.close(); } catch (SQLException e) {} }
        }
    }

}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    private DataSource dataSource;

    ...

    public void deleteAll() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            c = datasource.getConnection();

            StatementStrategy strategy = new DeleteAllStatement();
            ps = strategy.makePreparedStatement(c);

            ps.executeUpdate();
        } catch (SQLException e) {  
            throw e;
        } finally {
            if (ps != null) { try { ps.close(); } catch (SQLException e) {} }
            if (c != null) { try {c.close(); } catch (SQLException e) {} }
        }
    }    
}
```
{% endtab %}
{% endtabs %}

클라이언트와 컨텍스트의 클래스를 분리하진 않았지만 의존 관계와 책임 측면에서 볼 때 서로 잘 분리되어 있음을 알 수 있다. 특히 클라이언트가 컨텍스트에게 전략을 정해서 전달하는 것은 DI 구조라고 할 수 있다.

### add\(\) 메소드에 적용

이번에는 add\(\) 메소드에 적용해보자.

```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        Connection c = this.dataSource.getConnection();

        // 변하는 부분. 즉, 다른 클래스로 분리할 부분.
        PreparedStatement ps = c.prepareStatement(
            "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws SQLException {
        ...
    }

    public void deleteAll() throws SQLException {
       ...
    }    

    public int getCount() throws SQLException  {
        ...
    }
}
```

여기에서 변하는 부분인 PreparedStatement 코드를 AddStatement 클래스를 만들어 옮긴다.

```java
public class AddStatement implements StatementStrategy {
    // makePreparedStatement()에서 사용할 유저 정보를 저장할 변수
    User user;

    // 생성자를 통해 user 오브젝트를 제공받는다.
    public AddStatement(User user) {
        this.user = user;
    }   

    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)");

        // 제공받은 user로 정보를 가져온다.
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        return ps;
    }
}
```

이제 클라이언트인 `UserDao`의 `add()`는 user 정보를 생성자로 전달해줘야 한다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        // 생성자로 user를 전달한다.
        StatementStrategy st = new AddStatement(user);
        jdbcContextWithStatementStrategy(st);
    }

    public User get(String id) throws SQLException {
        ...
    }

    public void deleteAll() throws SQLException {
       ...
    }    

    public int getCount() throws SQLException  {
        ...
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        Connection c = this.dataSource.getConnection();

        PreparedStatement ps = c.prepareStatement(
            "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws SQLException {
        ...
    }

    public void deleteAll() throws SQLException {
       ...
    }    

    public int getCount() throws SQLException  {
        ...
    }
}
```
{% endtab %}
{% endtabs %}

### 마이크로 DI

DI의 가장 중요한 개념은 제3자를 통해 두 오브젝트 사이를 유연하게 만드는 것이다. 이 개념만 적용하면 구조나 관계는 다양하게 만들 수 있다.

DI는 의존 관계인 두 오브젝트와 서로의 관계를 다이나믹하게 설정해주는 오브젝트 팩토리\(DI 컨테이너\), 이를 사용하는 클라이언트 사이에서 일어난다.

하지만 때로는 원시적인 전략 패턴 구조에 따라 클라이언트가 오브젝트 팩토리의 역할을 같이 할 수도 있다. 또는 클라이언트와 전략이 합쳐질 수도 있으며 클라이언트, DI로 연결된 두 오브젝트 모두가 한 클래스에 담길 수도 있다. 이 경우 DI는 아주 작은 코드와 메소드에서 나타나기도 한다.

이렇게 DI의 장점을 단순화해서 IoC 컨테이너 없이 만드는 것을 `마이크로 DI`라고 한다. 코드에 의한 DI라는 의미로 수동 DI라고 하기도 한다.

