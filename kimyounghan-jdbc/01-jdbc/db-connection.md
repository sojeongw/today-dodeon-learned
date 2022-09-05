# 데이터베이스 연결

{% tabs %} {% tab title="ConnectionConst.java" %}

```java
// 상수로만 쓰고 객체 생성을 막기 위해 abstract를 사용한다.
public abstract class ConnectionConst {

    // 외부에서 가져다 써야 하므로 public이 된다.
    public static final String URL = "jdbc:h2:tcp://localhost/~/test";
    public static final String USERNAME = "sa";
    public static final String PASSWORD = "";

}
```

{% endtab %} {% tab title="DBConnectionUtil.java" %}

```java

@Slf4j
public class DBConnectionUtil {

    public static Connection getConnection() {
        try {
            Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
            log.info("get connection = {}, class = {}", connection, connection.getClass());

            return connection;
        } catch (SQLException e) {
            throw new IllegalStateException(e);
        }
    }
}
```

{% endtab %}{% tab title="DBConnectionUtilTest.java" %}

```java

@Slf4j
class DBConnectionUtilTest {

    @Test
    void connection() {
        Connection connection = DBConnectionUtil.getConnection();
        Assertions.assertThat(connection).isNotNull();
    }
}
```

{% endtab %} {% endtabs %}

- 데이터베이스에 연결하려면 JDBC가 제공하는 DriverManager.getConnection(..) 를 사용하면 된다.
- 이렇게 하면 라이브러리에 있는 데이터베이스 드라이버를 찾아서 해당 드라이버가 제공하는 커넥션을 반환해준다.
- 여기서는 H2 데이터베이스 드라이버가 작동해서 실제 데이터베이스와 커넥션을 맺고 그 결과를 반환한다.

```text
get connection = conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class = class org.h2.jdbc.JdbcConnection
```

- h2 전용 커넥션임을 확인할 수 있다.
- 이 커넥션은 표준 인터페이스인 `java.sql.Connection`을 구현하고 있다.

## JDBC DriverManager 연결 이해

![](../../.gitbook/assets/kimyounghan-jdbc/01/스크린샷%202022-09-17%20오후%203.55.14.png)

- JDBC는 `java.sql.Connection` 표준 커넥션 인터페이스를 정의한다.
- H2 데이터베이스 드라이버는 JDBC Connection 인터페이스를 구현한 `org.h2.jdbc.JdbcConnection` 구현체를 제공한다.

![](../../.gitbook/assets/kimyounghan-jdbc/01/스크린샷%202022-09-17%20오후%203.55.20.png)

DriverManager는 라이브러리에 등록된 DB 드라이버를 관리하고 커넥션을 획득한다.

1. 커넥션을 호출한다.
    - DriverManager.getConnection()
2. DriverManager가 라이브러리에 등록된 드라이버 목록을 자동으로 인식한다.
    - 드라이버에게 순서대로 정보를 넘겨 커넥션을 획득할 수 있는지 묻는다.
        - URL, 이름, 비밀번호 등
    - 각 드라이버는 URL 정보를 체크해 본인이 처리할 수 있는지 확인한다.
        - url이 `jdbc:h2`로 시작하면 h2 드라이버 본인이 처리할 수 있으므로 실제 DB에 연결해 커넥션을 반환한다.
        - 같은 url인데 다른 DB 드라이버가 먼저 실행되면 처리할 수 없다는 결과를 반환하고 다음 드라이버에게 넘어간다.
3. 이렇게 찾은 커넥션 구현체가 클라이언트에 반환된다.