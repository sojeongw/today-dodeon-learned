# JUnit 적용하기

테스트를 하면서 가장 불편했던 건 매번 테스트를 실행할 때마다 DB 데이터를 삭제하는 것이었다. 이 과정을 잊는다면 `add()` 메소드 실행중에 중복 에러가 뜰 것이다.

이렇게 별도의 준비 작업 때문에 테스트가 실패하기도 하고 성공하기도 한다면 좋은 테스트라고 할 수 없다. 테스트는 `항상` 동일한 결과를 내야 한다.

## deleteAll\(\)과 getCount\(\) 추가

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    // 1장에서 건너뛴 XML 파트에서 변경된 부분
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    // USER 테이블의 모든 레코드를 삭제해준다.
    public void deleteAll() throws SQLException {
        Connection c = dataSource.getConnection();

        PreparedStatement ps = c.prepareStatement("delete from users");
        ps.executeUpdate();

        ps.close();
        c.close();
    }

    // USER 테이블의 레코드 개수를 돌려준다.
    public int getCount() throws SQLException {
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

{% tab title="Before" %}
```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```
{% endtab %}
{% endtabs %}

이제 이 기능에 대한 테스트가 필요하다. 그런데 `deleteAll()` 과 `getCount()` 는 단독으로 테스트 하기가 애매하다. `USER` 테이블에 데이터를 수동으로 넣어서 확인해야 하므로 자동화된 테스트 방법이 아닌 것이다.

그래서 `addAndGet()` 테스트를 확장하는 것이 낫다. `addAndGet()`을 할 때 수동으로 테이블 내용을 삭제했어야 했으니 `deleteAll()로` 대체해보자.

`deleteAll()` 자체가 아직 검증되지 않았으므로 `getCount()`를 함께 적용해 확인해본다. 하지만 `getCount()`가 잘 동작하는지도 믿을 수 없다. 항상 0을 돌려주는 버그가 있을 수 있기 때문이다.

그래서 `getCount()`를 검증할 수 있도록 `add()` 메소드 실행 뒤에 `getCount()`를 다시 한번 체크해본다. `getCount()`가 제대로 동작한다면 `deleteAll()` 후에 0이 나오는 것도 바르게 동작한다고 생각할 수 있다.

{% tabs %}
{% tab title="After" %}
```java
import org.junit.runner.JUnitCore;                                  

public class UserDaoTest {
    public static void main(String[] args) {
        JUnitCore.main("user.dao.UserDaoTest");
    }

    @Test
    public void addAndGet() throws SQLException {
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);

        // 맨 처음에 USER 테이블 레코드를 다 지우고
        dao.deleteAll();
        // 0인지 확인한다.
        assertThat(dao.getCount(), is(0));

        User user = new User();
        user.setId("whiteship");
        user.setName("백기선");
        user.setPassword("married");

        // dao에 user를 추가하고
        dao.add(user);
        // 1인지 확인한다.
        assertThat(dao.getCount(), is(1));

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()));
        assertThat(user2.getPassword(), is(user.getPassword()));
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
import org.junit.runner.JUnitCore;                                  

public class UserDaoTest {
    public static void main(String[] args) {
        JUnitCore.main("user.dao.UserDaoTest");
    }

    @Test
    public void addAndGet() throws SQLException {
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);

        User user = new User();
        user.setId("whiteship");
        user.setName("백기선");
        user.setPassword("married");

        dao.add(user);

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()));
        assertThat(user2.getPassword(), is(user.getPassword()));
    }
}
```
{% endtab %}
{% endtabs %}

이전에는 테스트 전에 직접 DB에서 데이터를 삭제해야 했지만 이제 그런 번거로운 과정이 없어졌다. 테스트를 여러 번 실행해도 항상 동일한 결과를 얻을 수 있다.

다른 방법으로는 `addAndGet()` 테스트를 마치기 직전에 그동안 추가한 데이터를 다시 원상 복구 하는 것이다. `add()`로 호출한 데이터를 `deleteAll()`로 삭제하는 것이다.

이 방법도 나쁘진 않지만 `addAndGet()` 메소드 전 후로 같은 DB를 쓰는 작업이 있다면 문제가 생길 수 있으므로 _테스트 전_에 상태를 초기화 해주는 게 낫다.

테스트는 외부 환경이나 순서가 바뀌어도 항상 일관성 있는 결과가 보장되어야 함을 잊지 말자.

## 여러 개의 User 등록 후 테스트

앞서 `getCount()`에서 확인한 것은 `deleteAll()`을 실행했을 때 0인 경우와 `add()`를 한 번 실행한 뒤 1이 된 것뿐이다. 두 개 이상일 때는 어떻게 될 지 장담할 수 없다.

이번에는 `User`를 여러 번 등록하면서 `getCount()`를 확인해보자. 이 기능을 `addAndGet()`에 추가하는 건 좋지 않다. 테스트 메소드는 한 번에 한 가지 목적만 가져야 하기 때문이다.

JUnit은 아래의 조건만 지키면 한 클래스 안에 여러 개의 테스트 메소드를 넣을 수 있다.

* `@Test` 애노테이션
* `public` 접근 지정자
* `void` 리턴
* 파라미터 없음

USER 테이블을 지우고 `getCount()`를 했을 때 0인 것을 확인한 뒤, 3개의 정보를 추가하면서 `getCount()`가 제대로 동작하는지 확인해보자.

{% tabs %}
{% tab title="After" %}
```java
public class User {
    String id;
    String name;
    String password;

    // User 오브젝트를 여러 번 만들기 위해 생성자를 만든다.
    public User(String id, String name, String password) {
        this.id = id;
        this.name = name;
        this.password = password;
    }

    // 자바 빈 규약을 따르는 클래스에 생성자를 추가했을 때는
    // 파라미터가 없는 디폴트 생성자도 함께 정의해야 한다.
    public User() {
    } 

    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class User {
    String id;
    String name;
    String password;

    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
}
```
{% endtab %}
{% endtabs %}

`User`를 수정했다면 이제 `getCount()`에 대한 새로운 테스트를 만들어보자.

```java
public class UserDaoTest {
    @Test
    public void addAndGet() throws SQLException {
        ...
    }

    @Test
    public void count() throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext ("applicationContext.xml");

    // 새로운 User 3개를 생성하고
    UserDao dao = context.getBean("userDao" , UserDao.class);
    User user1 = new User("gyumee", "박성철", "spring1");
    User user2 = new User("leegw700", "이길원", "spring2");
    User user3 = new User("bumjin", "박범진", "spring3");

    // 다 지워지는지 확인한다.
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    // 유저를 하나씩 추가할 때마다 제대로 count 되는지 확인한다.
    dao.add(user1);
    assertThat(dao.getCount(), is(1));

    dao.add(user2);
    assertThat(dao.getCount(), is(2));

    dao.add(user3);
    assertThat(dao.getCount(), is(3));
    }
}
```

`deleteAll()`을 불러 테이블 내용을 모두 삭제한 뒤, `getCount()`가 제대로 동작하는지 확인한다.

이때 주의할 점은 테스트 메소드인 `count()`와 `addAndGet()`의 순서는 랜덤하다는 것이다. 테스트가 실행 순서에 영향을 받으면 안된다. 모든 테스트는 실행 순서에 상관 없이 항상 동일한 결과를 내야한다.

## id 검증

이번엔 `addAndGet()` 메소드를 보완해보자. `get()`이 파라미터로 받는 id가 진짜 존재하는 사용자를 가져오는지 확인해야 한다.

`User`를 하나 더 추가해서 두 개의 `User`를 `add()`하고 각 `User`의 id가 정확한지 검증한다.

{% tabs %}
{% tab title="After" %}
```java
import org.junit.runner.JUnitCore;                                  

public class UserDaoTest {
    public static void main(String[] args) {
        JUnitCore.main("user.dao.UserDaoTest");
    }

    @Test
    public void addAndGet() throws SQLException {
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);

        // 중복되지 않는 값을 가진 두 개의 User를 만든다.
        User user1 = new User("gyumee", "박성철", "spring1");
        User user2 = new User("leegw700", "이길원", "spring2");

        dao.deleteAll();
        assertThat(dao.getCount(), is(0));

        // 두 User를 모두 add() 한다.
        dao.add(user1);
        dao.add(user2);
        // 2명의 User를 제대로 count 했는지 확인한다.
        assertThat(dao.getCount(), is(2));

        // user 데이터를 get 한 다음
        User userget1 = dao.get(user1.getId());
        // 해당 유저의 값과 일치하는지 확인한다.
        assertThat(userget1.getName(), is(user1.getName()));
        assertThat(userget1.getPassword(), is(user1.getPassword()));

        // user 데이터를 get 한 다음
        User userget2 = dao.get(user2.getId());
        // 해당 유저의 값과 일치하는지 확인한다.
        assertThat(userget2.getName(), is(user2.getName()));
        assertThat(userget2.getPassword(), is(user2.getPassword()));
    }

    @Test
    public void count() throws SQLException {
        ...
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
import org.junit.runner.JUnitCore;                                  

public class UserDaoTest {
    public static void main(String[] args) {
        JUnitCore.main("user.dao.UserDaoTest");
    }

    @Test
    public void addAndGet() throws SQLException {
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);

        dao.deleteAll();
        assertThat(dao.getCount(), is(0));

        User user = new User();
        user.setId("whiteship");
        user.setName("백기선");
        user.setPassword("married");

        dao.add(user);
        assertThat(dao.getCount(), is(1));

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()));
        assertThat(user2.getPassword(), is(user.getPassword()));
    }

    @Test
    public void count() throws SQLException {
        ...
    }
}
```
{% endtab %}
{% endtabs %}

만약 파라미터로 받은 id가 존재하지 않는다면 어떻게 해야할까? 이럴 땐 두 가지 방법이 있다.

* null과 같은 특별한 값 리턴
* id에 해당하는 정보를 찾을 수 없다고 예외 처리

각기 장단점이 있지만 여기서는 후자를 해볼 것이다. 예외 클래스는 스프링의 `EmptyResultDataAccessException`을 사용한다.

테스트는 `UserDao`의 `get()`을 호출했을 때 해당 id에 대한 결과가 없을 때 예외를 던지면 된다. 그런데 지금까지 써온 `assertThat()`은 예외가 발생할 경우 테스트는 `실패`로 간주되어 중단된다. 우리는 예외가 던져져야 테스트가 `성공`한 것이므로 다른 방법을 사용할 것이다.

{% tabs %}
{% tab title="After" %}
```java
import org.junit.runner.JUnitCore;                                  

public class UserDaoTest {
    // 테스트 중에 발생하길 원하는 예외 클래스를 써준다. 이 예외가 발생하지 않으면 테스트는 '실패'한다.
    @Test(expected=EmptyResultDataAccessException.class)
    public void getUserFailure() throws SQLException {
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

        UserDao dao = context.getBean("userDao", UserDao.class);
        dao.deleteAll();
        assertThat(dao.getCount(), is(0));

        // 여기에서 예외가 발생해야 한다. 예외가 발생하지 않으면 테스트가 '실패'한다.
        dao.get("unknown_id");
    }

    @Test
    public void addAndGet() throws SQLException {
       ...
    }

    @Test
    public void count() throws SQLException {
        ...
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
import org.junit.runner.JUnitCore;                                  

public class UserDaoTest {
    @Test
    public void addAndGet() throws SQLException {
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);

        dao.deleteAll();
        assertThat(dao.getCount(), is(0));

        User user = new User();
        user.setId("whiteship");
        user.setName("백기선");
        user.setPassword("married");

        dao.add(user);
        assertThat(dao.getCount(), is(1));

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()));
        assertThat(user2.getPassword(), is(user.getPassword()));
    }

    @Test
    public void count() throws SQLException {
        ...
    }
}
```
{% endtab %}
{% endtabs %}

이제 id 존재 여부에 따라 데이터를 반환할 수 있도록 `UserDao`의 `get()`을 수정한다. 기존 코드는 존재하지 않는 파라미터를 보내면 `EmptyResultDataAccessException`대신 `SQLException`을 보낼 것이기 때문이다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        ...
    }

    public User get(String id) throws SQLException {
        Connection c = this.dataSource.getConnection();
        PreparedStatement ps = c
                .prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();

        // 일단 User를 null로 초기화한다.
        User user = null;

        // 받은 id에 대한 결과 데이터가 있다면
        if(rs.next()) {
            // 그 정보에 대한 오브젝트를 만들어 반환한다.
            user = new User();
            user.setId(rs.getString("id"));
            user.setName(rs.getString("name"));
            user.setPassword(rs.getString("password"));
        }

        rs.close();
        ps.close();
        c.close();

        // 반환값이 없다면 우리가 계획한 예외를 던진다.
        if(user == null) throw new EmptyResultDataAccessException(1);

        return user;

    }

    public void deleteAll() throws SQLException {
        ...
    }

    public int getCount() throws SQLException {
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
        ...
    }

    public User get(String id) throws SQLException {
        Connection c = this.dataSource.getConnection();
        PreparedStatement ps = c
                .prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }

    public void deleteAll() throws SQLException {
        ...
    }

    public int getCount() throws SQLException {
        ...
    }
}
```
{% endtab %}
{% endtabs %}

이렇게 하면 `getUserFailure()` 뿐만 아니라 `get()`을 사용하는 `addAndGet()` 테스트도 성공하게 된다. 즉, `get()`으로 정상 데이터를 가져오는 테스트와 예외를 발생시키는 테스트 모두 성공한 것이다.

## 포괄적인 테스트

이 정도의 간단한 DAO 코드를 굳이 테스트해야 하는지 반문하는 사람이 있을 것이다. 하지만 이렇게 포괄적인 테스트를 만드는 것이 훨씬 안전하고 유용하다.

평소에는 잘 동작하는 것 같았는데 막상 어떤 상황에서 동작하지 않으면 원인을 찾기가 힘들다. 때로는 단순하고 간단한 테스트가 치명적인 실수를 피할 수 있게 해준다.

특히 주의할 점은 성공하는 테스트만 골라서 하는 행위다. 코드가 잘 돌아가는 케이스를 상상하며 짜다보니 문제가 될 만한 상황을 `교묘하게 잘 피해서` 테스트 하는 습성이 있다.

> 제 컴퓨터에서는 잘 되는데요.

개발자들이 곧잘 하는 변명이다. 이는 개발자가 예외적인 상황을 피하고 정상 케이스만 테스트 했다는 뜻이다. 그래서 우리는 항상 `부정적인 케이스`를 먼저 만드는 습관을 들여야 한다.

예를 들어 `get()`의 경우 존재하는 id에 대해 레코드를 가져오는지 테스트 하는 것도 중요하지만 존재하지 않는 id일 때 어떻게 반응할지 먼저 결정하고 개발하는 것이 좋다.

