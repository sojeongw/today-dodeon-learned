# 1장. 오브젝트와 의존관계

## 스프링과 객체지향 설계

스프링은 오브젝트를 생성하고, 관계를 맺고, 사용하고, 소멸하기까지의 과정에 기준을 제공하는 프레임워크다.

### 자바 빈

아래의 두 가지를 가지고 만들어진 오브젝트

* 디폴트 생성자: 파라미터가 없는 디폴트 생성자
* 프로퍼티: 자바 빈이 노출하는 이름을 가진 것. setter와 getter로 수정 또는 조회한다.

### 리팩토링

기존에 동작하는 방식은 그대로 두고 내부 구조를 변경해서 재구성하는 작업 또는 기술을 의미한다.

* 코드를 이해하기가 편리해진다.
* 변화에 효율적으로 대응할 수 있다.
* 생산성이 올라간다.
* 코드의 품질이 높아진다.
* 유지보수가 용이해진다.

참고도서

[리팩토링](https://book.naver.com/bookdb/book_detail.nhn?bid=7047630)

### 디자인 패턴

소프트웨어를 만들 때 자주 발생하는 문제를 해결하기 위해 사용할 수 있는 솔루션이다. 주로 객체지향적 설계 원칙을 따른다. `클래스 상속`과 `오브젝트 합성` 두 가지 구조로 정리된다.

참고도서 

[GoF의 디자인 패턴](https://book.naver.com/bookdb/book_detail.nhn?bid=8942623) 

[Head First Design Patterns](https://book.naver.com/bookdb/book_detail.nhn?bid=1882446)

### 객체 지향 설계 원칙\(SOLID\)

객체 지향의 특징을 잘 살릴 수 있는 원칙이다. 절대적인 기준이라고 보다는 예외는 있겠지만 대부분의 상황에 잘 들어맞는 가이드라인 같은 것이다. `디자인 패턴`이 특별한 상황에서 사용하는 좀 더 구체적인 솔루션이라면 `객체 지향 설계 원칙`은 좀 더 일반적인 설계 기준이라고 할 수 있다.

* SRP\(The Single Responsibility Principle\): 단일 책임 원칙
* OCP\(Open/Closed Principle\): 개방 폐쇄 원칙
* LSP\(Liskov Substitution Principle\): 리스코프 치환 원칙
* ISP\(Interface Segregation Principle\): 인터페이스 분리 원칙
* DIP\(Dependency Inversion Principle\): 의존관계 역전 원칙

참고도서 

[UML 실전에서는 이것만 쓴다](https://book.naver.com/bookdb/book_detail.nhn?bid=6439362) 

[소프트웨어 개발의 지혜](https://book.naver.com/bookdb/book_detail.nhn?bid=144677)

## 초난감 DAO

DB 커넥션과 statement 실행이 add\(\)와 get\(\)에 중복으로 들어가있다.

```java
public class UserDAO {

    public void add(User user){
        // DB 커넥션
        Connection c = DriverManager()...

        // statement 실행
        PreparedStatement ps = ...
    }

    public User get(String id){
         // DB 커넥션
         Connection c = DriverManager()...

         // statement 실행
         PreparedStatement ps = ...       
    }
}
```

### 관심사의 분리

개발자는 미래에 일어날 변화에 대비할 수 있는 코드를 짜야 한다. 최소한의 수정만으로 변화가 가능해야 한다. 관심이 같은 것은 같은 혹은 친한 객체로 모아 놓는다. 

여기서 관심사란 `DB와 연결할 커넥션을 어떻게 가져올 것인가`, `SQL 문장을 어떻게 실행할 것인가` 등과 같은 것들이다.

#### 중복 코드의 메소드 추출

소드 추출 기법: 중복된 코드를 독립적인 메소드로 만든다.

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

    private Connection getConnection(){
        // DB 커넥션
        Connection c = DriverManager()...
    }
}
```

#### 상속을 통한 확장

* 템플릿 메소드 패턴: 기본 골격을 담은 메소드. 변하지 않는 기능은 슈퍼 클래스에, 자주 변경하거나 확장할 기능은 서브 클래스에 만든다.  추상 클래스, 같은 패키지나 서브 클래스에서 이용할 수 있는 protected가 쓰인다. 서브 클래스에서 선택적으로 오버라이드 할 수 있는 메소드를 `hook 메소드`라고 한다. 
* 팩토리 메소드 패턴: 템플릿 메소드 패턴과 비슷한데 주로 인터페이스를 이용한다. 인터페이스이기 때문에 어떤 클래스를 실제로 구현해서 리턴할지는 알지 못하며 관심도 없다.  서브 클래스에서 결정할 수 있도록 미리 기본 코드를 짜놓은 메소드를 `팩토리 메소드`라고 하는데, 자바에서 오브젝트를 생성하는 `팩토리 메소드`와 혼동하지 않아야 한다.

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

#### 단점

* 다중 상속이 허용되지 않아 변경 사항이 있을 때 추가적인 상속을 적용할 수 없다.
* 슈퍼 클래스가 변경되면 모든 서브 클래스를 수정해야 한다.

### 클래스의 분리

상속이 아니라 완전히 독립된 클래스로 분리한다. 객체를 한 번만 만들어 놓고 계속 사용할 수 있다.

```java
// 이제 상속을 하지 않으므로 추상 클래스일 필요가 없다.
public class ConnectionMaker {
    public Connection makeConnection() {
        // DB 커넥션 코드
    }
}
```

```java
public class UserDAO {
    // 따로 클래스를 만들어서
    private ConnectionMaker connectionMaker;

    public UserDao() {
        // 생성자에서 인스턴스를 만들고
        connectionMaker = new ConnectionMaker();
    }

    // 각각의 메소드에서 사용한다.
    public void add(User user){
        Connection c = connectionMaker.makeConnection();
    }

    public User get(String id){
        Connection c = connectionMaker.makeConnection();
    }
}
```

#### 단점

분리한 클래스에 종속된다. 즉, 분리한 클래스를 변경하려면 사용하고 있는 클래스도 변경해줘야 한다. 예를 들어 D사에 맞추려면 `ConnectionMaker`를 D사 용으로 다시 만들어야 한다. 이 문제는 우리가 구체적으로 어떤 클래스를 사용할지 `알고 있기` 때문에 일어난다.

### 인터페이스 도입

#### 추상화

추상화란 공통점을 추려서 따로 분리하는 것이다. 두 개의 클래스가 서로 긴밀하게 연결되지 않도록 느슨한 연결고리를 만들어준다. 실제 구현한 클래스가 무엇인지는 `몰라도` 된다. 그저 해당 인터페이스 타입으로 넘겨주기만 하면 된다. 인터페이스로 사용할 수 있는 기능만 알면 되지, 어떻게 구현했는지는 알 필요가 없다.

```java
// DB 커넥션 부분을 인터페이스로 분리한다.
public interface ConnectionMaker {
    public Connection makeConnection() {
    }
}
```

```java
// 해당 인터페이스를 implements 한 다음
public class DConnectionMaker implements ConnectionMaker {

    // 인터페이스에 있는 메소드를 원하는 대로 구현한다.
    public Connection makeConnection {
    }
}
```

```java
public class UserDAO {
    // D사인지 N사인지 명시할 필요 없이 인터페이스만 가져온다.
    private ConnectionMaker connectionMaker;

    public UserDao() {
        // 생성자에서 인터페이스를 구현하는 클래스를 선언한다.
        // 문제 발생! 어쨌든 여기서도 D사인지 N사인지 수정해야 한다.
        connectionMaker = new DConnectionMaker();
    }

    public void add(User user){
        // 어떤 클래스를 구현하든 인터페이스를 통하는 것이므로 이 코드에는 변화가 없다.
        Connection c = connectionMaker.makeConnection();
    }

    public User get(String id){\
        Connection c = connectionMaker.makeConnection();
    }
}
```

### 관계 설정 책임의 분리

인터페이스를 사용하더라도, 어떤 클래스로 구현할지 명시해야 하기 때문에 여전히 완전하게 분리되지 않는다. UserDAO 안에 또 다른 관심사항이 존재하는 것이다.

#### 클라이언트

여기서 클라이언트는 오브젝트를 사용하는 또 다른 오브젝트를 의미한다. UserDAO를 사용하는 클라이언트 오브젝트에 해당 관심사를 분리한다. 클라이언트에서 인터페이스를 구현하는 클래스를 명시하고 파라미터로 UserDAO에 보내면 파라미터는 인터페이스라는 조건만 충족하면 되므로 UserDAO에서 구체적인 클래스를 알 필요가 없어진다.

```java
// 클라이언트 클래스를 생성한다.
public class UserDaoTest {
    public static void main(String[] args) {

        // 여기서 구체적인 클래스를 명시해준다.
        // 만약 N사로 바뀌더라도 이 부분만 변경해주면 된다.
        ConnectionMaker connectionMaker = new DConnectonMaker();

        // 생성자에 파라미터로 넘겨준다.
        UserDao dao = new UserDao(connectionMaker);
    }
}
```

```java
public class UserDAO {
    private ConnectionMaker connectionMaker;

    // 생성자로 인터페이스를 받아온다. 인터페이스 타입만 맞추면 되므로 어떤 클래스가 구체적으로 구현되어 있는지는 알 필요가 없다.
    public UserDao(ConnectionMaker connectonMaker) {
        this.connectionMaker = connectonMaker();
    }

    public void add(User user){
        Connection c = connectionMaker.makeConnection();
    }

    public User get(String id){\
        Connection c = connectionMaker.makeConnection();
    }
}
```

### 원칙과 패턴

#### 개방 폐쇄 원칙

클래스나 모듈은 `확장`에는 열려있고, `변경`에는 닫혀있어야 한다.

#### 높은 응집도

모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 의미다. 변화가 일어날 때 해당 모듈에서 바꿔야할 부분이 많다는 뜻이기도 하다. 불필요하거나 직접 관련이 없는 외부의 관심과 책임은 얽혀있지 않다.

#### 낮은 결합도

책임과 관심사가 다른 오브젝트 또는 모듈과 느슨하게 연결되어 있는 것이다. 꼭 필요한 최소한의 방법만 간접적으로 제공하고 나머지는 서로 독립적인 형태다. 이렇게 되면 변화에 대응하는 속도가 높아지고 구성이 깔끔해진다. 확장하기에도 편리하다.

#### 전략 패턴

변경이 필요한 부분은 인터페이스로 분리시키고 필요할 때 바꿔서 사용할 수 있게 하는 디자인 패턴이다. 컨텍스트\(UserDAO\)를 사용하는 클라이언트\(UserDaoTest\)가 컨텍스트가 사용하는 전략\(DConnectionMaker\)를 생성자 등으로 제공하는 것이다.

