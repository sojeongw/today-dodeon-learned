# 스프링 AOP

아래와 같은 코드가 있다고 생각해보자.

 ```java
class A {
    method a() {
        pre();
        aaa();
        post();
    }
}
class B {
    method b() {
        pre();
        bbb();
        post();
    }
}
class C {
    method c() {
        pre();
        bbb();
        post();
    }
}
```

모든 클래스는 `pre()`와 `post()`를 공통으로 가지고 있다. 이때 이 두 메서드에 수정이 필요하다면 세 클래스 모두 건드려야 한다. 

```java
class A {
    method a() {
        aaa();
    }
}
class B {
    method b() {
        bbb();
    }
}
class C {
    method c() {
        bbb();
    }
}
class PrePro {
    method prePro(JoinPoint point) {
        pre();
        point.execute();
        post();
    }
}
```

그래서 공통된 메서드는 별도의 클래스로 빼낸다. 이게 바로 AOP다.

## AOP 구현 방법

### 컴파일

> A.java ----> (AOP) ----> A.class

A.java 파일을 컴파일 하면 A.class 파일이 생긴다. 컴파일을 하는 과정에서 AOP를 끼워넣는 것이다. `AspectJ`가 이 방법을 지원한다.

### 바이트코드 조작

> A.java ----> A.class ----> (AOP) ----> 메모

클래스 로더가 A.class 파일을 읽어오면서 메모리에 올릴 때 조작하는 방법이다. 

클래스 파일의 코드 상엔 아무것도 없는데 클래스를 로딩하는 시점에 메모리 상에서 추가하고 싶은 코드가 들어가는 것이다. 역시 `AspectJ`가 제공한다.

### 프록시 패턴

스프링 AOP가 사용하는 방법이다. 빈이 등록될 때 자동으 프록시가 만들어진다.

```java
public class toreTest {
    @Test
    public void testPay() {
        // 프록시를 넣어준다.
        Payment cashPerf = new CashPerf();
        // 프록시를 쓰지 않았다면 아래처럼 Cash를 그대로 썼을 것이다.
        // Payment cash = new Cash();

        Store store = new Store(cashPerf);
        store.buySomething(100);
    }
}

public class Store {

    Payment payment;

    public Store(Payment payment) {
        this.payment = payment;
    }

    public void buySomething(int amount) {
        payment.pay(amount);
    }
}

public class Cash implements Payment{
    @Override
    public void pay(int amount) {
        System.out.println(amount + " 현금 결제");
    }
}

// 일종의 프록시
public class CashPerf implements Payment{
    Payment cash = new Cash();

    @Override
    public void pay(int amount) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        System.out.println("Start");

        cash.pay(amount);

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
    }
}
```

```text
Start
100 현금 결제
StopWatch '': running time = 138590 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
000138590  100%  
```

우리는 `Store`나 `Cash`를 건드리지 않고도 `Stopwatch`를 이용한 부가 기능을 사이에 끼워넣었다. 스프링에 빗대자면, `Cash`를 빈으로 등록해놨지만 내가 만들고 싶은 프록시가 자동으로 생성되어 클라이언트는 `Cash` 대신에 프록시를 사용한다.

```java
public interface OwnerRepository extends Repository<Owner, Integer> {
	@Transactional(readOnly = true)
	Collection<Owner> findByLastName(@Param("lastName") String lastName);
}
```

원래 JDBC를 쓸 때면 코드 앞 뒤로 `setAutoCommit()`과 커밋, 롤백 코드가 들어갔다. 하지만 `@Transactional`이 붙으면 `OwnerRepository` 객체 타입의 프록시가 새로 생성되어 그 코드를 생략할 수 있다.

**Reference**

[프록시 패턴](https://refactoring.guru/design-patterns/proxy)