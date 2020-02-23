# 예외 전환

예외 전환의 목적은 두 가지로 나뉜다.

* 런타임 예외로 포장해서 굳이 필요하지 않은 catch/throws를 줄인다.
* 로우레벨의 예외를 좀 더 의미있고 추상화된 예외로 바꾼다.

`DataAccessException`은 런타임 예외로 `SQLException`을 포장해서, 복구 불가능한 내용은 애플리케이션 레벨에서 신경쓰지 않도록 해준다.

또, `SQLException`에서 다루기 힘든 상세한 예외 정보를 의미 있고 일관성 있는 예외로 전환해서 추상화 한다.

## JDBC의 한계

JDBC는 DB 접근 방식을 추상화한 API를 정의해서 각기 다른 DB 업체들이 표준에 맞게 드라이버를 제공하도록 해준다. 하지만 여전히 자유롭게 사용할 수 없는 부분이 있다.

### 비표준 SQL

SQL은 어느정도 표준화되어있지만 비표준 문법과 기능도 존재한다. 대용량 데이터 처리의 성능을 높이기 위해 최적화 기법을 적용하는 등 비표준 SQL을 사용하게 되면 특정 DB에 종속된다.

### 호환성 없는 SQLException의 DB 에러 정보

DB가 다르면 에러의 종류와 원인도 제각각이다. JDBC는 이런 다양한 에러를 `SQLException` 하나에 묶어버린다. `e.getErrorCode()` 로 고유의 정보를 확인할 수 있지만 DB에 따라 정확하지 않은 값이 들어가있는 경우도 있다.

따라서 SQL 상태 코드를 믿고 결과를 파악하는 것은 위험하며 `SQLException`만으로 DB에 독립적인 코드를 작성하는 것은 불가능하다.

## DB 에러 코드 매핑

`SQLException`에 담긴 상태 코드는 신뢰성이 없지만, DB 업체별로 제공하는 전용 에러코드는 좀 더 정확하다. 이 에러 코드를 참고해서 예외 원인을 파악할 수 있다.

스프링은 `DataAccessException` 이라는 런타임 예외를 정의하고 있는데 `SQLException`을 대체할 수 있으며 `BadSqlGrammarException` 등 세분화 된 서브 클래스를 제공한다.

![](../../.gitbook/assets/toby/20200204111558.png)

하지만 DB 마다 에러 코드가 제각각이므로 위와 같이 분류해서 스프링이 정의한 예외 클래스와 매핑한 테이블을 만들어둔다.

JdbcTemplate은 `SQLException`을 단지 `DataAccessException`으로 포장하는 것이 아니라 각자의 에러 코드를 그 아래의 클래스 중 하나로 매핑해준다.

이렇게 DB별로 미리 준비해둔 리스트를 참고해서 적절한 예외 클래스를 선택하기 때문에 DB가 달라져도 같은 종류의 에러는 동일한 예외를 받을 수 있다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    // 이제 따로 중복 키 에러를 분류할 필요가 없다.
    // DB를 변경하더라도 JdbcTemplate을 이용해 동일한 예외를 던져준다.
    public void add() throws DuplicateKeyException {
        // JDBCTemplate을 이용해 User를 add하는 코드
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class DuplicateUserIdException extends RuntimeException {
    public DuplicateUserIdException(Throwable cause) {
        super(cause);
    }
}

public class UserDao {
    public void add() throws DuplicateUserIdException {
        try {
        }
        catch(SQLException e) {
            if(e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
                // 예외 전환
                throw new DuplicateUserIdException(e);
            else
                // 예외 포장
                throw new RuntimeException(e);
        }
    }
}
```
{% endtab %}
{% endtabs %}

에러가 발생했을 때 애플리케이션에서 직접 정의한 예외를 발생시킬 수도 있다. 이때는 아래처럼 예외를 전환해주면 된다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDao {
    // DuplicateUserIdException이라는 애플리케이션 레벨의 체크 예외로 전환해 던진다.
    public void add() throws DuplicateUserIdException {
        try {
            // JDBCTemplate을 이용해 User를 add하는 코드
        }
        catch(DuplicateKeyException e) {
            // 로그를 남기는 등의 필요한 작업 코드

            // 예외를 전환할 때는 원인이 되는 예외를 중첩하는 것이 좋다.
            throw new DuplicateUserIdExcepton(e);
        }
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
    public void add() throws DuplicateKeyException {
        // JDBCTemplate을 이용해 User를 add하는 코드
    }
}
```
{% endtab %}
{% endtabs %}

`DuplicateKeyException`을 `DuplicateUserIdExcepton`으로 전환해 던지고 있다.

## DataAccessException의 계층 구조

`DataAccessException`은 의미가 같은 예외라면 JDBC든, Hibernate든 일관된 예외가 나오도록 만들어준다. 데이터 액세스 기술에 독립적으로 추상화된 예외를 제공하는 것이다.

## DAO 인터페이스와 구현의 분리
### 장점

DAO를 굳이 따로 만드는 이유는 아래와 같다.

- 데이터 액세스 로직을 분리할 수 있음
- 전략 패턴을 적용해 구현 방법을 변경하며 사용할 수 있음

DAO를 사용하는 오브젝트는 DAO가 그 안에서 어떤 데이터 액세스 기술을 쓰는지 신경 쓸 필요가 없다. 그냥 사용하기만 하면 된다. 따라서 인터페이스로 구체적인 클래스 정보와 구현 방법은 감추고 DI로 제공되는 것이 좋다.

### 단점

그런데 DAO의 구체적인 사용 기술과 코드는 전략 패턴과 DI을 이용해 클라이언트로부터 감출 수 있지만 메소드 선언의 예외 정보가 문제될 수 있다.

```java
public interface UserDao {
    public void add(User user);
}
```

DAO의 데이터 액세스 기술 API가 예외를 던지기 때문에 이런 식으로 메소드 선언은 불가능하다. 

```java
public interface UserDao throws SQLException {
    public void add(User user);
}
```

만약 JDBC API를 사용하는 DAO의 구현 클래스라면 SQLException을 던진다. 따라서 throws를 반드시 넣어서 선언해야 한다.

```java
public void add(User user) throws PersistentException;  // JPA
public void add(User user) throws HibernateException;   // Hibernate
public void add(User user) throws JdoException;    // JDO
```

그런데 데이터 액세스 기술 API는 자기만의 예외를 던진다. 따라서 인터페이스로 추상화를 해도 예외 때문에 메소드를 선언한 부분이 제각각이 된다.

단순히 모든 예외를 받아주는 `throws Exception`으로 선언하면 굉장히 무책임한 해결 방법이다.

### 런타임 예외로 포장하기

기술에 따라 예외 처리 방법이 달라진다면 결국 클라이언트는 DAO 기술에 의존하게 된다. 다행히 JPA, Hibernate, JDO는 `SQLException` 같은 체크 예외 대신 런타임 예외를 사용한다. 따라서 throws에 선언하지 않아도 된다.

문제는 JDBC를 사용하는 DAO다. 이 경우 DAO 메소드 안에서 런타임 예외로 포장해서 던지면 된다. 모든 `SQLException`을 런타임 예외로 포장하면 다시 아래처럼 선언할 수 있다.

```java
public interface UserDao {
    public void add(User user);
}
```

하지만 과연 이것으로 충분할까? 대부분의 데이터 액세스 예외는 복구 불가능하거나 할 필요가 없다. 그렇다고 다 무시해야 하는 것도 아니다.

### DataAccessException 계층 구조 활용하기

그래서 스프링은 자바의 다양한 데이터 액세스 기술의 예외를 추상화해서 `DataAccessException`에 정리해놓았다.

#### 데이터 액세스 기술을 부정확하게 사용했을 경우

예를 들어 JDBC의 `BadSqlGrammarException`과 하이버네이트의 `HibernateQueryException`은 잘못된 작성으로 발생하는 오류다. 

이때 스프링은 `InvalidDataAccessResourceUsageException` 타입의 예외로 던져주어 시스템 레벨의 예외를 처리해 개발자에게 빨리 알려준다.

#### 낙관적인 락킹(Optimistic locking)이 발생할 경우

`낙관적인 락킹`은 같은 정보를 두 명 이상의 사용자가 동시에 조회하고 순차적으로 업데이트 할 때, 뒤늦게 업데이트 한 것이 먼저 업데이트 한 것을 덮어쓰지 않도록 막아주는 기능이다.

이런 예외가 생기면 사용자에게 안내 메시지를 띄우고 다시 시도할 수 있도록 해야한다. 이것 역시 기술마다 다르게 발생하는 예외를 `DataAccessException`의 서브 클래스인 `ObjectOptimisticLockingFailureException`으로 통일할 수 있다.

![](../../.gitbook/assets/toby/screenshot%202020-02-21%20오후%2010.56.52.png)

만약 직접 낙관적인 락킹 기능을 구현하고 싶다면? 앞서 언급한 `ObjectOptimisticLockingFailureException`의 슈퍼 클래스인 `OptimisticLockingFailureException`를 상속해서 `JdbcOptimisticLockingFailureException`이라는 새로운 클래스를 정의할 수 있다. 이 역시 기술에 상관없이 만들 수 있다.

따라서 런타임 예외 전환과 함께 `DataAccessException` 예외 추상화를 적용하면 데이터 액세스 기술에 독립적인 DAO를 만들 수 있다.

## 기술에 독립적인 UserDao
### 인터페이스 적용

이제 `UserDao` 클래스를 인터페이스와 구현 클래스로 분리해보자. 

인터페이스에는 보통 I를 접두어로 붙이거나 구현 클래스에 특징을 붙이는 경우가 있는데 후자의 방법을 사용할 것이다. 사용자 처리 DAO는 `UserDao`, JDBC를 이용해 구현한 클래스는 `UserDaoJdbc`다.

{% tabs %}
{% tab title="After" %}
```java
public interface UserDao {
    void add(User user);
    User get(String id);
    List<User> getAll();
    void deleteAll();
    int getCount();
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}
	
	private JdbcTemplate jdbcTemplate;
	
	private RowMapper<User> userMapper = 
		new RowMapper<User>() {
				public User mapRow(ResultSet rs, int rowNum) throws SQLException {
				User user = new User();
				user.setId(rs.getString("id"));
				user.setName(rs.getString("name"));
				user.setPassword(rs.getString("password"));
				return user;
			}
    };
	
	public void add(final User user) {
		this.jdbcTemplate.update("insert into users(id, name, password) values(?,?,?)",
						user.getId(), user.getName(), user.getPassword());
	}

	public User get(String id) {
		return this.jdbcTemplate.queryForObject("select * from users where id = ?",
				new Object[] {id}, this.userMapper);
	} 

	public void deleteAll() {
		this.jdbcTemplate.update("delete from users");
	}

	public int getCount() {
		return this.jdbcTemplate.queryForInt("select count(*) from users");
	}

	public List<User> getAll() {
		return this.jdbcTemplate.query("select * from users order by id",this.userMapper);
	}
}
```
{% endtab %}
{% endtabs %}

`setDataSource()` 메서드는 인터페이스에 추가하면 안 된다. `UserDao`의 구체적인 구현 방법에 따라 변경될 수 있고, 굳이 `UserDao`를 사용하는 클라이언트가 알고 있을 필요가 없는 정보이기 때문이다.

기존 `UserDao` 클래스는 `UserDaoJdbc`로 이름을 변경하고 `UserDao` 인터페이스를 구현한다.

{% tabs %}
{% tab title="After" %}
```java
public interface UserDaoJdbc implements UserDao {
    public void setDataSource(DataSource dataSource) {
  		this.jdbcTemplate = new JdbcTemplate(dataSource);
  	}
  	
    private JdbcTemplate jdbcTemplate;
    
    private RowMapper<User> userMapper = 
        new RowMapper<User>() {
                public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                User user = new User();
                user.setId(rs.getString("id"));
                user.setName(rs.getString("name"));
                user.setPassword(rs.getString("password"));
                return user;
            }
    };
    
    public void add(final User user) {
        this.jdbcTemplate.update("insert into users(id, name, password) values(?,?,?)",
                        user.getId(), user.getName(), user.getPassword());
    }
    
    public User get(String id) {
        return this.jdbcTemplate.queryForObject("select * from users where id = ?",
                new Object[] {id}, this.userMapper);
    } 
    
    public void deleteAll() {
        this.jdbcTemplate.update("delete from users");
    }
    
    public int getCount() {
        return this.jdbcTemplate.queryForInt("select count(*) from users");
    }
    
    public List<User> getAll() {
        return this.jdbcTemplate.query("select * from users order by id",this.userMapper);
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDao {
public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}
	
	private JdbcTemplate jdbcTemplate;
	
	private RowMapper<User> userMapper = 
		new RowMapper<User>() {
				public User mapRow(ResultSet rs, int rowNum) throws SQLException {
				User user = new User();
				user.setId(rs.getString("id"));
				user.setName(rs.getString("name"));
				user.setPassword(rs.getString("password"));
				return user;
			}
    };
	
	public void add(final User user) {
		this.jdbcTemplate.update("insert into users(id, name, password) values(?,?,?)",
						user.getId(), user.getName(), user.getPassword());
	}

	public User get(String id) {
		return this.jdbcTemplate.queryForObject("select * from users where id = ?",
				new Object[] {id}, this.userMapper);
	} 

	public void deleteAll() {
		this.jdbcTemplate.update("delete from users");
	}

	public int getCount() {
		return this.jdbcTemplate.queryForInt("select count(*) from users");
	}

	public List<User> getAll() {
		return this.jdbcTemplate.query("select * from users order by id",this.userMapper);
	}
}
```
{% endtab %}
{% endtabs %}

내용은 바꿀 필요 없이 클래스 이름만 변경하면 끝이 난다.

### 테스트 보완

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserDaoTest {
	@Autowired 
	private UserDao dao; 
	
	private User user1;
	private User user2;
	private User user3;
	
	@Before
	public void setUp() {
        ...
	}
	
	@Test 
	public void andAndGet() {		
        ...
	}

	@Test(expected=EmptyResultDataAccessException.class)
	public void getUserFailure() throws SQLException {
        ...
	}

	
	@Test
	public void count() {
        ...
	}
	
	@Test
	public void getAll()  {
        ...
	}

	private void checkSameUser(User user1, User user2) {
        ...
	}

}
```

이전 테스트 코드에는 `dao`가 `UserDao`로 되어있다. 이것을 굳이 `UserDaoJdbc`로 바꿀 필요는 없다. `@Autowired`는 스프링 컨텍스트 내에서 정의된 빈을 보고 인스턴스 변수에 주입할 수 있는 타입의 빈을 찾아준다.

여기서는 `UserDaoJdbc`가 `UserDao`를 구현했기 때문에, 변수 `dao`에 `UserDao` 타입인 `UserDaoJdbc` 빈을 넣어준다. 

상황에 따라 변수 `dao`를 `UserDaoJdbc` 타입으로 선언할 수도 있다. 단지 DAO 기능이 동작하는 데만 관심있다면 전자가 낫고, 특정 기술을 사용한 `UserDao` 구현 내용에 초점을 맞추려면 후자가 낫다.

![](../../.gitbook/assets/toby/screenshot%202020-02-23%20오후%205.32.38.png)

인터페이스와 구현 클래스를 분리했으므로 데이터 액세스의 구체적인 기술을 클라이언트 `UserDaoTest`에 DI로 적용시키고 있다.

이제 중복된 값이 들어갈 때 제대로 예외를 처리하는지 확인해보자.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserDaoTest {
	@Autowired 
	private UserDao dao; 
	
	private User user1;
	private User user2;
	private User user3;
	
	@Before
	public void setUp() {
        ...
	}
	
    ...

    @Test(expected=DataAccessException.class)
    public void duplicateKey() {
        dao.deleteAll();
        
        dao.add(user1);
        // 강제로 같은 사용자를 두 번 등록해 예외를 발생시킨다.
        dao.add(user1);
    }

}
```

`DataAccessException`이 발생하면 테스트는 성공이다. 만약 이 부분을 빼고 테스트를 실행하면 테스트가 실패하면서 아래와 같은 로그가 찍힌다.

```text
org.springframework.dao.DuplicateKeyException: PreparedStatementCallback; SQL [insert into users(id, name, password) values(ι7，7)]; Duplicate entry ’gyumee' for key 1; 

nested exception is com.mysql.jdbc.exceptions.jdbc4.MySQLlntegrityConstraintViolationException: Duplicate entry ’ gyumee ’ for key 1
```

`DataAccessException`의 서브 클래스인 `DuplicateKeyException` 예외가 발생했다. 따라서 `expected` 내용을 `DuplicateKeyException`로 바꾸면 좀더 정확한 예외를 발생시킬 수 있다.

### DataAccessException 활용 시 주의 사항

하지만 `DuplicateKeyException`은 JDBC를 이용하는 경우에만 발생한다. JPA나 하이버네이트 등은 `SQLException`에 담긴 DB 에러 코드와 달리 세분화된 예외를 제공하지 않기 때문이다.

`DataAccessException`이 어느정도 추상화된 공통 예외로 변환해주긴 하지만 이런 근본적이 한계 때문에 사용에 주의를 기울여야 한다. 미리 학습 테스트를 만들어서 실제 전환되는 예외의 종류를 확인해두는 것이다.

만약 기술 종류와 상관없이 동일한 예외를 얻고 싶다면 직접 예외를 정의한 뒤, `add()` 메서드에 상세한 예외 전환을 해준다.

스프링은 `SQLException`을 `DataAccessException`으로 전환하는 다양한 방법을 제공하는데, 그 중 DB 에러 코드를 활용해보자.

`SQLErrorCodeSQLExceptionTranslator`는 에러 코드 변환에 필요한 DB 종류를 알아내기 위해 `DataSource`를 필요로 한다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserDaoTest {
	@Autowired UserDao dao; 

    // DB 종류를 알아내기 위해 사용한다.
	@Autowired DataSource dataSource;
	
	private User user1;
	private User user2;
	private User user3;
	
	@Before
	public void setUp() {
        ...
	}
	
    ...

	@Test
	public void sqlExceptionTranslate() {
		dao.deleteAll();
		
		try {
            // DuplicateKeyException을 강제로 발생시킨다.
			dao.add(user1);
			dao.add(user1);
		}
        // 중첩된 예외. JDBC API에서 처음 발생한 SQLException을 갖고 있다.
		catch(DuplicateKeyException ex) {
            // 중첩되어 있는 SQLException을 가져온다.
			SQLException sqlEx = (SQLException)ex.getRootCause();
            // 주입받은 datasource로 SQLErrorCodeSQLExceptionTranslator 오브젝트를 만든다.
			SQLExceptionTranslator set = new SQLErrorCodeSQLExceptionTranslator(this.dataSource);

            // SQLException 타입 sqlEx 변수를 넣어 translate 메소드로 호출한다.
            // 그러면 SQLExcpetion을 DataAccessException 타입의 예외로 변환한다.
			DataAccessException transEx = set.translate(null, null, sqlEx);

            // 변환된 DataAccessException 타입의 예외가 DuplicateKeyException 타입인지 확인한다.
			assertThat(transEx, is(DuplicateKeyException.class));
		}
	}
}
```