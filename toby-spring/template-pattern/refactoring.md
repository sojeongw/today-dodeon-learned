# 내부 클래스로 리팩토링 하기

현재까지 만든 구조는 DAO 메소드 마다 새로운 StatementStrategy 구현 클래스를 만들어야 한다. 또한, add()의 User처럼 추가적으로 필요한 정보가 있으면 전략 클래스에 생성자와 인스턴스 변수를 굳이 만들어줘야 한다.

## 중첩 클래스

다른 클래스 내부에 정의한 클래스를 중첩 클래스라고 한다. 중첩 클래스에는 두 종류가 있다.

### 스태틱 클래스(static class)

독립적인 오브젝트로 만들 수 있다.

### 내부 클래스(inner class)

자신이 정의된 클래스의 오브젝트 안에서만 만들어진다. scope에 따라 다시 세 가지로 분류된다.

#### 멤버 내부 클래스(member inner class)

멤버 필드처럼 오브젝트 레벨에서 정의된다.

#### 로컬 클래스(local class)

메소드 레벨에서 정의된다.

#### 익명 내부 클래스(anonymous inner class)

이름을 갖지 않으며 선언된 위치에 따라 범위가 정해진다.

## 로컬 클래스

StatementStrategy 전략 클래스를 매번 별도의 파일로 만들지 않고 `UserDao`의 내부 클래스로 만드는 방법이 있다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
	private DataSource dataSource;
    		
    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    // 내부 클래스의 외부에 있는 변수를 사용하려면 final로 선언해줘야 한다.
    public void add(final User user) throws SQLException {
        // 원래 따로 만들었던 AddStatement 클래스를 그대로 add() 메소드 안으로 가져왔다.
        class AddStatement implements StatementStrategy {
            // 인스턴스 변수와 생성자는 user를 따로 받아오지 않아도 되므로 삭제한다.
        
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)");
        
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());

                return ps;
            }
        }

        // 이제는 user를 파라미터로 전달하지 않아도 된다.
        StatementStrategy st = new AddStatement();
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
{% endtabs %}

`AddStatement` 클래스를 add() 메소드 안에 그대로 넣었다. 즉, 로컬 클래스의 형태다. 로컬 변수를 선언하듯이 사용하면 되며, 선언된 메소드 내에서만 사용할 수 있다.

이렇게 하면 어차피 `AddStatement` 클래스는 `add()`에서만 사용되므로 클래스 파일 하나를 줄일 수 있고 로직도 한 번에 볼수 있다.

또한, 클래스가 내부에 선언되어 있어 이제는 User 정보를 생성자로 전달해줄 필요가 없다.

## 익명 내부 클래스

```java
new 인터페이스이름() { 클래스 내용 };
```

익명 내부 클래스는 이름이 없는 클래스다. 클래스 선언과 오브젝트 생성이 합쳐져 있다. 상속할 클래스나 인터페이스를 생성자 대신 사용한다. 

이름이 없으니 자신의 클래스 타입을 가질 수 없고 구현한 인터페이스 타입의 변수에만 저장할 수 있다. 클래스를 재사용할 일이 없고 구현한 인터페이스 타입으로만 쓸 때 유용하다.

### add()에 적용하기

`AddStatement` 클래스를 `add()`에서만 사용한다면 굳이 `AddStatement`라는 이름을 쓰지 않아도 된다. 

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
	private DataSource dataSource;
    		
    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(final User user) throws SQLException {
        // 익명 내부 클래스로 선언한다.
        // 인터페이스 StatementStrategy 타입으로만 받고 있다.
        StatementStrategy st = new StatementStrategy() {
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)");
        
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());

                return ps;
            }
        };

        StatementStrategy st = new AddStatement();
        jdbcContextWithStatementStrategy(st);
    }
    ...
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

    public void add(final User user) throws SQLException {
        class AddStatement implements StatementStrategy {
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)");
        
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());

                return ps;
            }
        }

        StatementStrategy st = new AddStatement();
        jdbcContextWithStatementStrategy(st);
    }
    ...
}
```
{% endtab %}
{% endtabs %}

그런데 익명 내부 클래스의 오브젝트는 딱 한 번만 사용한다. 굳이 변수에 담아둘 필요가 없으니 컨텍스트 메소드에서 바로 생성해보자.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
	private DataSource dataSource;
    		
    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void add(final User user) throws SQLException {
        // 한 번만 쓰는 거니까 굳이 인터페이스 타입에 담지 않고
        // jdbcContextWithStatementStrategy() 메소드의 파라미터에서 바로 생성한다.
        jdbcContextWithStatementStrategy(new StatementStrategy() {
             public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                 PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)");
         
                 ps.setString(1, user.getId());
                 ps.setString(2, user.getName());
                 ps.setString(3, user.getPassword());
 
                 return ps;
             }
         });
        // 이제 이곳에서 전략을 만들고 컨텍스트로 보낼 필요가 없다.
    }
    ...
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

    public void add(final User user) throws SQLException {
        StatementStrategy st = new StatementStrategy() {
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)");
        
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());

                return ps;
            }
        };
        
        StatementStrategy st = new AddStatement();
        // 이전에는 이렇게 컨텍스트에 전략을 담아 보냈다.
        jdbcContextWithStatementStrategy(st);
    }
    ...
}
```
{% endtab %}
{% endtabs %}

### deleteAll()에 적용하기

같은 방식으로 deleteAll()도 적용해보자.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    ...

    public void deleteAll() throws SQLException {
        jdbcContextWithStatementStrategy(
            new StatementStrategy() {
                public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                    return c.preparedStatement("delete form users");    
                }
            }
        );
    }
    ...
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    ...

    public void deleteAll() throws SQLException {
        StatementStrategy st = new DeleteAllStatement();
        jdbcContextWithStatementStrategy(st);
    }

	public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
		...
	}
}
```
{% endtab %}
{% endtabs %}