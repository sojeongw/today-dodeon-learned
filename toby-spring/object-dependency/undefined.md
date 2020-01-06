# 오브젝트 팩토리

사실 이전에 만든 클라이언트 `UserDaoTest`는 원래 `UserDao`가 잘 동작하는지 테스트하기위한 것이었다.

```java
public class UserDaoTest {
    public static void main(String[] args) {
    
        ConnectionMaker connectionMaker = new DConnectonMaker();
        UserDao dao = new UserDao(connectionMaker);
    }
}
```

하지만 `ConnectionMaker` 인터페이스를 구현한 클래스를 독립시키기 위해 3번 라인을 추가하면서 또 다른 책임까지 맡게 되었다. 지금까지 해왔던 것처럼 책임과 관심사에 따라 분리해보자.

## 팩토리

객체의 생성 방법을 결정하고 그 오브젝트를 돌려주는 역할을 한다.오브젝트를 생성하는 부분과 실제 사용하는 부분을 분리하는 목적으로 사용한다.

{% tabs %}
{% tab title="After" %}
```java
public class DaoFactory {
    public UserDao userDao() {
        // 옮기고
        ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        
        return userDao;
    }
}

...

public class UserDaoTest {
    public static void main(String[] args) {
        // 옮긴 것을 사용한다.
        UserDao dao = new DaoFactory.userDao();
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDaoTest {
    public static void main(String[] args) {
    
        ConnectionMaker connectionMaker = new DConnectonMaker();
        UserDao dao = new UserDao(connectionMaker);
    }
}
```
{% endtab %}
{% endtabs %}

`UserDaoTest`가 갖고 있던 `ConnectionMaker`와 `UserDao` 생성 코드를 `DaoFactory`로 옮겼다. 이제 `UserDao`가 어떻게 생성되는지 신경쓰지 않고도 테스트를 위해 활용할 수있게 되었다.

### 설계도로서의 팩토리

| UserDao, ConnectionMaker | DaoFactory |
| :---: | :---: |
| 데이터 로직, 기술 로직 구현 | 오브젝트를 구성하고 서로의 관계 정의 |
| 컴포넌트 | 설계 |

`UserDao`와 `ConnectionMaker`가 애플리케이션의 데이터 로직과 기술 로직을 담당하고 있고, `DaoFactory`는 이러한 오브젝트를 구성하고 서로의 관계를 정의하는 역할을 한다.

이제 N사와 D사가 각각 다른 connection을 원해도 `DaoFactory`만 수정하면 해결할 수 있다. `컴포넌트`역할의 오브젝트와 `설계도/구조`를 결정하는 오브젝트를 분리함으로써 얻는 장점이다.

## 오브젝트 팩토리의 활용

만약 `DaoFactory`에서 `UserDao`가 아니라 다른 `Dao`를 넣고 싶다면 어떻게 될까?

```java
public class DaoFactory {
    public UserDao userDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        
        return userDao;
    }
    
    public AccountDao accountDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        AccountDao accountDao = new AccountDao(connectionMaker);
        
        return accountDao;
    }
    
    public MessageDao messageDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        MessageDao messageDao = new MessageDao(connectionMaker);
        
        return messageDao;
    }
}
```

`ConnectionMaker`를 생성하는 코드가 메소드마다 중복으로 나타나게 된다.이렇게 하면 `ConnectionMaker`의 구현 클래스가 바뀌면 모든 메소드를 수정해야 한다.

중복을 해결하려면 분리하는 것이 가장 좋은 방법이다. `ConnectionMaker`의 구현 클래스를 설정하고 그 오브젝트를 만드는 부분만 별도로 뽑아낸다.

{% tabs %}
{% tab title="After" %}
```java
public class DaoFactory {
    public UserDao userDao() {
        UserDao userDao = new UserDao(connectionMaker());
        
        return userDao;
    }
    
    public AccountDao accountDao() {
        AccountDao accountDao = new AccountDao(connectionMaker());
        
        return accountDao;
    }
    
    public MessageDao messageDao() {
        MessageDao messageDao = new MessageDao(connectionMaker());
        
        return messageDao;
    }
    
    // 실제 connection을 구현하는 클래스를 생성하는 부분만 따로 빼낸다.
    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class DaoFactory {
    public UserDao userDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        
        return userDao;
    }
    
    public AccountDao accountDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        AccountDao accountDao = new AccountDao(connectionMaker);
        
        return accountDao;
    }
    
    public MessageDao messageDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        MessageDao messageDao = new MessageDao(connectionMaker);
        
        return messageDao;
    }
}
```
{% endtab %}
{% endtabs %}

이제 `connectionMaker()` 한 부분만 수정하면 모든 `DaoFactory` 메소드에 적용된다.



