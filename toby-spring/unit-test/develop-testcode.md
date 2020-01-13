# 일관성 있는 테스트

테스트를 하면서 가장 불편했던 건 매번 테스트를 실행할 때마다 DB 데이터를 삭제하는 것이었다. 이 과정을 잊는다면 `add()` 메소드 실행중에 중복 에러가 뜰 것이다.

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

이 방법도 나쁘진 않지만 `addAndGet()` 메소드 전 후로 같은 DB를 쓰는 작업이 있다면 문제가 생길 수 있으므로 *테스트 전*에 상태를 초기화 해주는 게 낫다.

테스트는 외부 환경이나 순서가 바뀌어도 항상 일관성 있는 결과가 보장되어야 함을 잊지 말자.

## 여러 개의 User 등록 후 테스트

앞서 `getCount()`에서 확인한 것은 `deleteAll()`을 실행했을 때 0인 경우와 `add()`를 한 번 실행한 뒤 1이 된 것뿐이다. 두 개 이상일 때는 어떻게 될 지 장담할 수 없다.

이번에는 `User`를 여러 번 등록하면서 `getCount()`를 확인해보자. 이 기능을 `addAndGet()`에 추가하는 건 좋지 않다. 테스트 메소드는 한 번에 한 가지 목적만 가져야 하기 때문이다.

JUnit은 아래의 조건만 지키면 한 클래스 안에 여러 개의 테스트 메소드를 넣을 수 있다.

- `@Test` 애노테이션
- `public` 접근 지정자
- `void` 리턴
- 파라미터 없음

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
    ...
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