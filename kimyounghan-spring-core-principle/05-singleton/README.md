# 웹 애플리케이션과 싱글턴

스프링은 태생이 기업용 온라인 서비스 기술을 지원하기 위함이었다. 대부분의 스프링 애플리케이션은 웹 애플리케이션이며 웹은 보통 여러 고객이 동시에 요청한다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-10%20오후%208.03.16.png)

직접 만들었던 DI 컨테이너는 요청을 할 때마다 새로운 인스턴스를 만들어 반환한다.

{% tabs %} {% tab title="SingletonTest.java" %}

```java
public class SingletonTest {

  @Test
  @DisplayName("스프링 없는 순수한 DI 컨테이너")
  void pureContainer() {
    AppConfig appConfig = new AppConfig();

    MemberService memberService1 = appConfig.memberService();
    MemberService memberService2 = appConfig.memberService();

    System.out.println("memberService1 = " + memberService1);
    System.out.println("memberService2 = " + memberService2);

    assertThat(memberService1).isNotSameAs(memberService2);
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-10%20오후%208.08.02.png)

스프링 없는 순수한 DI 컨테이너로 테스트 하면, 두 인스턴스가 다른 참조값이라는 것을 알 수 있다. 생각해보면 `memberService`에서는 또 저장소 인스턴스를 만드니 총
4개가 생성되는 것이다. 그럼 요청이 올 때마다 JVM 메모리에 계속 객체가 쌓이게 되므로 **메모리 낭비**가 심하다.

객체가 딱 하나만 생성되고 공유되도록 설계하면 된다. 그것이 바로 싱글턴 패턴이다.

## 싱글턴 패턴

- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴
- private 생성자를 이용해 외부에서 임의로 2개 이상의 new 키워드를 사용하지 못하도록 막아야 한다.

{% tabs %} {% tab title="SingletonService.java" %}

```java
public class SingletonService {

  // 자기 자신을 private static으로 만들어 하나만 올라가도록 한다.
  // jvm이 뜨는 시점에 초기화하면서 new 한 번으로 생성해서 가지고 있는다.
  private static final SingletonService instance = new SingletonService();

  // public으로 열어서 객체 인스턴스가 필요하면 이 static 메서드를 통해서면 조회하도록 허용한다.
  public static SingletonService getInstance() {
    return instance;
  }

  // 외부 클래스에서 new SingletonService()로 임의로 만드는 것을 막기 위해 private 생성자를 만든다.
  private SingletonService() {
  }

  public void logic() {
    System.out.println("싱글턴 객체 로직 호출");
  }

}

```

{% endtab %} {% tab title="SingletonTest.java" %}

```java
public class SingletonTest {

  @Test
  @DisplayName("싱글턴 패턴을 적용한 객체 사용")
  void singletonServiceTest() {
    SingletonService singletonService1 = SingletonService.getInstance();
    SingletonService singletonService2 = SingletonService.getInstance();

    System.out.println("singletonService1 = " + singletonService1);
    System.out.println("singletonService2 = " + singletonService2);

    // same과 equal은 다르다.
    // same는 == 비교, equal은 equals() 비교와 같다.
    assertThat(singletonService1).isSameAs(singletonService2);
  }
}
```

{% endtab %} {% endtabs %}

1. static 영역에 객체 instance를 하나만 미리 생성해서 올려둔다.
2. `getInstance()`를 통해 항상 같은 인스턴스를 반환한다.
3. 생성자를 private으로 막아 외부에서 new 키워드로 객체 인스턴스가 생성되는 일을 막는다.

생성을 딱 한번만 하도록 만들고 오직 `getInstance()`를 통해서만 조회할 수 있도록 만들면 싱글턴이 된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-10%20오후%208.25.05.png)

테스트 코드를 돌리면 같은 참조 값이 나오는 것을 볼 수 있다.

> 싱글턴 패턴의 구현 방법은 여러가지가 있다. 여기서는 객체를 미리 생성해두는 가장 단순하고 안전한 방법을 선택했다.

## 싱글턴의 문제점

싱글턴 패턴을 적용하면 고객의 요청이 올 때마다 생성하지 않고 이미 만들어진 것을 공유해서 사용하므로 효율적이다. 하지만 문제점도 가지고 있다.

- 싱글턴 패턴을 구현하는 코드 자체가 많이 들어간다.
    - private static 변수 선언, getInstance(), 생성자 등
- 의존 관계상 클라이언트가 구체 클래스에 의존해 DIP와 OCP를 위반한다.
    - `구체클래스.getInstance()` 방식으로 가져와야 한다.
- 이미 딱 받아놓은 변수이기 때문에 유연하기 테스트하기 어렵다.
- private 생성자 때문에 자식 클래스를 만들기 어렵다.
- 그래서 안티 패턴으로 불리기도 한다.

## 싱글턴 컨테이너

스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도 객체 인스턴스를 싱글턴으로 관리한다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%201.12.28.png)

이전에 살펴봤듯, 컨테이너는 객체를 하나만 생성해서 관리한다.

스프링 컨테이는 싱글턴 컨테이너 역할을 한다. 이렇게 싱글턴 객체를 생성하고 관리하는 기능을 `싱글턴 레지스트리`라고 한다.

덕분에 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글턴으로 유지할 수 있다.

{% tabs %} {% tab title="SingletonTest.java" %}

```java
public class SingletonTest {

  @Test
  @DisplayName("스프링 컨테이너와 싱글턴")
  void springContainer() {
    AnnotationConfigApplicationContext ac =
        new AnnotationConfigApplicationContext(AppConfig.class);

    MemberService memberService1 = ac.getBean("memberService", MemberService.class);
    MemberService memberService2 = ac.getBean("memberService", MemberService.class);

    System.out.println("memberService1 = " + memberService1);
    System.out.println("memberService2 = " + memberService2);

    assertThat(memberService1).isSameAs(memberService2);
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-10%20오후%208.38.29.png)

지저분한 코드 없이도 싱글턴으로 관리되는 것을 확인했다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-10%20오후%208.35.02.png)

스프링 컨테이너 덕분에 고객의 요청이 올 때마다 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.

## 싱글턴 방식의 주의점

싱글턴 패턴이든 스프링 같은 싱글톤 컨테이너를 사용하든 한 인스턴스를 생성해서 공유하는 싱글턴 방식은 여러 클라이언트가 하나를 공유하기 때문에 싱글턴 객체의 상태를 유지하도록(
stateful) 설계하면 안된다.

### 무상태(stateless) 설계

- 특정 클라이언트에 의존적인 필드가 있으면 안된다.
- 특정 클라이언트가 값을 변경할 수 있는 필드가 없어야 한다.
- 가급적 읽기만 가능해야 한다.
- 필드 대신 자바에서 공유되지 않는 지역 변수, 파라미터, ThreadLocal 등을 사용해야 한다.
- 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다.

{% tabs %} {% tab title="StatefulService.java" %}

```java
public class StatefulService {

  // 상태를 유지하는 필드
  private int price;

  public void order(String name, int price) {
    System.out.println("name = " + name + ", price = " + price);
    this.price = price;

  }

  public int getPrice() {
    return price;
  }

  @Test
  void statefulServiceSingleton() {
    AnnotationConfigApplicationContext ac =
        new AnnotationConfigApplicationContext(TestConfig.class);

    StatefulService statefulService1 = ac.getBean(StatefulService.class);
    StatefulService statefulService2 = ac.getBean(StatefulService.class);

    // ThreadA: A 사용자 10000원 주문
    statefulService1.order("userA", 10000);

    // ThreadB: B 사용자 20000원 주문
    statefulService2.order("userB", 20000);

    // ThreadA: A 사용자 주문 금액 조회
    int price = statefulService1.getPrice();
    System.out.println("price = " + price);

    assertThat(statefulService1.getPrice()).isEqualTo(20000);
  }

  static class TestConfig {

    @Bean
    public StatefulService statefulService() {
      return new StatefulService();
    }
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-10%20오후%208.49.19.png)

우리가 기대한 건 10000원이지만, 결과는 20000원이 나왔다.

`statefulService1`과 `statefulService2`는 같은 인스턴스이기 때문에 값이 바뀌어버리는 것이다.

{% tabs %} {% tab title="After" %}

```java
public class StatefulService {

  // price 변수를 삭제한다.

  public int order(String name, int price) {
    System.out.println("name = " + name + ", price = " + price);

    // 리턴한다.
    return price;
  }

  @Test
  void statefulServiceSingleton() {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(
            TestConfig.class);

    StatefulService statefulService1 = ac.getBean(StatefulService.class);
    StatefulService statefulService2 = ac.getBean(StatefulService.class);

    // 반환한 price를 사용한다.
    int price1 = statefulService1.order("userA", 10000);
    int price2 = statefulService2.order("userB", 20000);

    System.out.println("price1 = " + price1);
    System.out.println("price2 = " + price2);
  }
}
```

{% endtab %} {% tab title="Before" %}

```java
public class StatefulService {

  private int price;

  public void order(String name, int price) {
    System.out.println("name = " + name + ", price = " + price);
    this.price = price;

  }

  public int getPrice() {
    return price;
  }

  @Test
  void statefulServiceSingleton() {
    AnnotationConfigApplicationContext ac =
            new AnnotationConfigApplicationContext(TestConfig.class);

    StatefulService statefulService1 = ac.getBean(StatefulService.class);
    StatefulService statefulService2 = ac.getBean(StatefulService.class);

    statefulService1.order("userA", 10000);
    statefulService2.order("userB", 20000);

    int price = statefulService1.getPrice();
    System.out.println("price = " + price);
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-10%20오후%208.58.25.png)

이렇게 상태를 저장하지 않도록 해야 한다.
