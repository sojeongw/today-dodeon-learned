# 일관성 있는 테스트

테스트를 하면서 가장 불편했던 건 매번 테스트를 실행할 때마다 DB 데이터를 삭제하는 것이었다. 이 과정을 잊는다면 add() 메소드 실행중에 중복 에러가 뜰 것이다.

이렇게 별도의 준비 작업 때문에 테스트가 실패하기도 하고 성공하기도 한다면 좋은 테스트라고 할 수 없다. 테스트는 `항상` 동일한 결과를 내야 한다.

## deleteAll()과 getCount() 추가

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    private ConnectionMaker connectionMaker;

    public void setConnectionMaker(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
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
    private ConnectionMaker connectionMaker;

    public void setConnectionMaker(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
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

이 방법도 나쁘진 않지만 `addAndGet()` 메소드 전 후로 같은 DB를 쓰는 작업이 있다면 문제가 생길 수 있으므로 <u>테스트 전</u>에 상태를 초기화 해주는 게 낫다.

테스트는 외부 환경이나 순서가 바뀌어도 항상 일관성 있는 결과가 보장되어야 함을 잊지 말자.