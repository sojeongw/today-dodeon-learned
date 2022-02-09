# 3장. 템플릿

## 개방 폐쇄 원칙\(OCP\)

확장에는 자유롭게 열려 있고 변경에는 굳게 닫혀 있다는 객체지향 설계의 핵심 원칙이다.

코드를 보면 어떤 부분은 수정하면서 다양하게 확장하려고 하고, 어떤 부분은 고정되어 변하지 않으려고 한다. 이때 각기 특성이 다른 부분을 구분하고 독립적으로 변경될 수 있게 하는 것이다.

## 템플릿

이 중에서 변경이 거의 일어나지 않고 일정한 패턴으로 유지되는 부분을 변경되는 부분에서 독립시키는 방법이다.

## 리소스 반환과 close\(\)

`Connection`이나 `PreparedStatement`의 `close()`는 리소스를 반환하는 메소드다. 이 둘은 보통 pool 방식으로 운영된다. 미리 정해진 풀 안에 제한된 수의 리소스\(Connection, Statement\)를 만들어두고 필요할때 할당하고 반환하면 다시 풀에 넣는다.

서버 환경에서는 요청이 매우 많아 매번 새로운 리소스를 생성하는 대신 풀에 미리 만들어둔 리소스를 돌려가며 사용하는 것이 훨씬 유리하다. 대신 사용한 리소스는 빠르게 반환해야 한다.

그렇지 않으면 풀에 있는 리소스가 고갈되고 문제가 발생한다. 이를 방지하기 위해 `close()` 메소드는 리소스를 풀로 다시 돌려주는 역할을 한다.

## UserDAO의 템플릿 적용

UserDAO는 DB 커넥션에 대한 예외 상황을 처리하지 않았다. DB 커넥션은 반드시 예외 처리가 필요하다. 중간에 어떤 예외에 발생해도 사용한 리소스를 반드시 반환해야 하기 때문이다.

```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        ...
    }

    public User get(String id) throws SQLException {
        ...
    }

    public void deleteAll() throws SQLException {
        Connection c = dataSource.getConnection();

        // 여기서 예외가 발생하면 바로 메소드 실행이 중단된다.
        PreparedStatement ps = c.prepareStatement("delete from users");
        ps.executeUpdate();

        // 예외가 발생하면 리소스를 반환하지 못하게 된다.
        ps.close();
        c.close();
    }    

    public int getCount() throws SQLException  {
        ...
    }
}
```

일반적으로 서버는 제한된 개수의 DB 커넥션을 만들고 재사용 가능한 풀로 관리한다. 매번 `getConnection()`으로 가져온 커넥션을 `close()`로 명시해서돌려줘야 다음 커넥션 요청 시 재사용할 수 있다.

하지만 오류가 나서 반환되지 못한 `Connection`이 계속 쌓이면 어느 순간 커넥션 풀에 여유가 없어지고 리소스가 모자라 심각한 오류와 함께 서버가 중단될 수 있다.

그래서 JDBC는 어떤 상황에서도 가져온 리소스를 반환하도록 try/catch/finally 사용을 권장한다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        ...
    }

    public User get(String id) throws SQLException {
        ...
    }

    public void deleteAll() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            // 예외가 발생할 수 있는 코드를 모두 try 블록으로 묶는다.
            c = datasource.getConnection();
            ps = c.prepareStatement("delete from users");
            ps.executeUpdate();
        } catch (SQLException e) {  // 예외가 발생하면 처리하는 블록
            throw e;
        } finally {     // 예외 발생 여부와 상관 없이 무조건 실행된다.
            if(ps != null){
                try {
                    ps.close();
                } catch (SQLException e) {  
                    // close() 메소드에서도 예외가 발생할 수 있다.
                    // 만약 여기서 예외를 잡아주지 않으면 close() 하지 못하고 메소드를 빠져나갈 수 있다.
                }
            }
            if(c != null) {
                try {
                    // 커넥션 반환
                    c.close();
                } catch (SQLException e) {

                }
            }
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

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        ...
    }

    public User get(String id) throws SQLException {
        ...
    }

    public void deleteAll() throws SQLException {
        Connection c = dataSource.getConnection();

        PreparedStatement ps = c.prepareStatement("delete from users");
        ps.executeUpdate();

        ps.close();
        c.close();
    }    

    public int getCount() throws SQLException  {
        ...
    }
}
```
{% endtab %}
{% endtabs %}

try 블록을 사용했다면 반드시 `close()` 호출로 가져온 리소스를 반환해야 한다. 이때 `close()` 메소드의 호출 시점이 중요하다.

### getConnection\(\)에서 DB 커넥션을 가져올 때

일시적으로 DB나 네트워크에 문제가 있거나 다른 예외가 생겼다면 `ps`와 `c` 둘 다 null 상태다. `null`일 때 `close()`를 호출하면 `NullPointerException`이 발생하므로 `close()`를 호출해서는 안된다.

### PreparedStatement를 생성하다가 예외가 발생했을 때

이때는 `c`가 이미 커넥션 객체를 가지고 있어 `close()` 호출이 가능하지만 `ps`는 그렇지 않다.

### ps를 실행하다가 예외가 발생했을 때

`c`와 `ps` 모두 `close()` 메소드를 호출해줘야 한다.   
 따라서, `finally`에서는 반드시 `c`와 `ps`가 `null`이 아닌지 먼저 확인한 후에 `close()`를 호출해야 한다. close\(\) 또한 `SQLException`이 날 수 있으므로 try/catch로 처리한다. `close()` 실패 시 특별히 해줄 수 있는 조치는 없다.

이미 `deleteAll()`에 `SQLException`이 처리되어 있지만, 만약 try/catch를 쓰지 않는다면 `ps.close()`를 한 뒤 `c.close()`가 실행되지 않는 경우가 발생할 수도 있다.

또한, 현재는 catch에서 예외를 던지는 일 밖에 없어서 빼버릴 수도 있지만 보통 로그를 남기거나 부가적인 추가 작업이 있을 수 있으니 일단 만들어두는 것이 좋다.   
 이제 `getCount()`를 바꿔보자. JDBC 조회 기능은 좀 더 복잡하다. `Connection`, `PreparedStatement` 외에 `ResultSet`이 필요하기 때문이다. `ResultSet` 또한 `close()`가 호출되도록 만든다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        ...
    }

    public User get(String id) throws SQLException {
        ...
    }

    public void deleteAll() throws SQLException {
        ...
    }    

    public int getCount() throws SQLException  {
        Connection c = null;
        PreparedStatement ps = null;
        // 조회를 위한 ResultSet 추가
        ResultSet rs = null;

        try {
            c = dataSource.getConnection();
            ps = c.preparedStatement("select count(*) from users");

            // rs도 SQLException이 발생할 수 있으니 try에 넣는다.
            rs = ps.executeQuery();
            rs.next();
            return rs.getInt();
        } catch (SQLException e) {

        } finally {
            if(rs != null) {
                try {
                    // rs를 닫아준다.
                    // close()는 만든 순서의 반대로 하는 것이 원칙이다.
                    rs.close();
                } catch (SQLException e) {

                }
            }
            if(ps != null) {
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
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        ...
    }

    public User get(String id) throws SQLException {
        ...
    }

    public void deleteAll() throws SQLException {
        ...
    }    

    public int getCount() throws SQLException  {
        Connection c = dataSource.getConnection();

        PreparedStatement ps = c.prepareStatement("select count(*) from users");

        ResultSet rs = ps.executeQuery();
        rs.next();
        int count = rs.getInt(1);

        rs.close();
        ps.close();
        c.close();

        return count;
    }
}
```
{% endtab %}
{% endtabs %}

