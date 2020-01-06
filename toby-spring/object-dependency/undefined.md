# 관심사의 분리

개발자는 미래에 일어날 변화에 대비할 수 있는 코드를 짜야 한다. 최소한의 수정만으로 변화가 가능해야 한다. 관심이 같은 것은 같은 혹은 친한 객체로 모아 놓는다. 

여기서 관심사란 `DB와 연결할 커넥션을 어떻게 가져올 것인가`, `SQL 문장을 어떻게 실행할 것인가` 등과 같은 것들이다.

## 중복 코드의 메소드 추출

메소드 추출 기법: 중복된 코드를 독립적인 메소드로 만든다.

{% tabs %}
{% tab title="After" %}
```java
public class UserDAO {

    public void add(User user){
        // 메소드만 호출
        Connection c = getConnection();
    }

    public User get(String id){
         // 메소드만 호출
         Connection c = getConnection();     
    }

    // 새로 만든 메소드
    private Connection getConnection(){
        // DB 커넥션
        Connection c = DriverManager()...
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDAO {
    private ConnectionMaker connectionMaker;

    public UserDao() {
        connectionMaker = new ConnectionMaker();
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

## 상속을 통한 확장

### 템플릿 메소드 패턴

기본 골격을 담은 메소드. 변하지 않는 기능은 슈퍼 클래스에, 자주 변경하거나 확장할 기능은 서브 클래스에 만든다. 

추상 클래스, 같은 패키지나 서브 클래스에서 이용할 수 있는 protected가 쓰인다. 서브 클래스에서 선택적으로 오버라이드 할 수 있는 메소드를 `hook 메소드`라고 한다.

### 팩토리 메소드 패턴

템플릿 메소드 패턴과 비슷한데 주로 인터페이스를 이용한다. 인터페이스이기 때문에 어떤 클래스를 실제로 구현해서 리턴할지는 알지 못하며 관심도 없다. 

서브 클래스에서 결정할 수 있도록 미리 기본 코드를 짜놓은 메소드를 `팩토리 메소드`라고 하는데, 자바에서 오브젝트를 생성하는 `팩토리 메소드`와 혼동하지 않아야 한다.

{% tabs %}
{% tab title="After" %}
```java
public abstract class UserDAO {

    public void add(User user){
        Connection c = getConnection();
    }

    public User get(String id){\
         Connection c = getConnection();     
    }

    // 추상 메소드로 수정
    private abstract Connection getConnection(){
        // 구체적인 내용은 서브 클래스가 담당한다.
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDAO {

    public void add(User user){
        Connection c = getConnection();
    }

    public User get(String id){
         Connection c = getConnection();     
    }

    private Connection getConnection(){
        Connection c = DriverManager()...
    }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Subclass" %}
```java
public class NUserDAO extends UserDAO {
    // N사만의 connection 구현
    public Connection getConnection() {
    }
}

public class DUserDAO extends UserDAO {
    // D사만의 connection 구현
    public Connection getConnection() {
    }
}
```
{% endtab %}
{% endtabs %}

### 단점

* 다중 상속이 허용되지 않아 변경 사항이 있을 때 추가적인 상속을 적용할 수 없다.
* 슈퍼 클래스가 변경되면 모든 서브 클래스를 수정해야 한다.

