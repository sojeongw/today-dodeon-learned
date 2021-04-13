# 프로토타입 스코프

싱글턴 스코프는 빈을 조회할 때 스프링 컨테이너가 항상 같은 인스턴스의 스프링 빈을 반환한다. 반면 프로토타입 스코프는 스프링 컨테이너가 항상 새로운 인스턴스를 생성해서 반환한다.

## 싱글턴 빈 요청

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.22.00.png)

1. 싱글턴 스코프의 빈을 스프링 컨테이너에 요청한다.
2. 스프링 컨테이너는 본인이 관리하는 스프링 빈을 반환한다.
3. 같은 요청이 와도 항상 같은 인스턴스의 스프링 빈을 반환한다.

## 프로토타입 빈 요청

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.22.33.png)

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.22.40.png)

1. 프로토타입 스코프의 빈을 스프링 번테이너에 요청한다.
2. 스프링 컨테이너는 요청 받은 시점에 프로토타입 빈을 생성하고 필요한 의존 관계를 주입한다.
3. 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.
4. 이후 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환한다.

핵심은 스프링 컨테이너가 프로토타입 빈을 생성하고, 의존 관계를 주입하고, 초기화하는 것까지만 처리한다는 것이다.

클라이언트에 빈을 반환한 뒤에는 스프링 컨테이너가 생성된 프로토타입 빈을 관리하지 않는다. 프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에 있다.
그래서 `@PreDestory`같은 종료 메서드가 호출되지 않는다.

{% tabs %} {% tab title="SingletonTest.java" %}

```java
public class SingletonTest {

  @Test
  void singleBeanFind() {
    // 빈을 직접 넣어주면 해당 빈을 자동으로 컴포넌트 스캔해서 빈으로 등록한다.
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(
        SingletonBean.class);

    SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
    SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);

    System.out.println("singletonBean1 = " + singletonBean1);
    System.out.println("singletonBean2 = " + singletonBean2);

    assertThat(singletonBean1).isSameAs(singletonBean2);

    ac.close();
  }

  // 그래서 여기에 @Component가 없어도 스프링 빈으로 등록되는 것이다.
  @Scope("singleton")
  static class SingletonBean {

    @PostConstruct
    public void init() {
      System.out.println("SingletonBean.init");
    }

    @PreDestroy
    public void destroy() {
      System.out.println("SingletonBean.destroy");
    }
  }

}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.38.39.png)

싱글턴은 당연히 동일한 인스턴스가 출력된다. `init`과 `destroy`도 올바르게 호출되었다.

{% tabs %} {% tab title="PrototypeTest.java" %}

```java
public class PrototypeTest {

  @Test
  void prototypeBeanFind() {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(
        PrototypeBean.class);

    System.out.println("find prototypeBean1");
    PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
    PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

    System.out.println("prototypeBean1 = " + prototypeBean1);
    System.out.println("prototypeBean2 = " + prototypeBean2);

    assertThat(prototypeBean1).isNotSameAs(prototypeBean2);
    ac.close();
  }

  @Scope("prototype")
  static class PrototypeBean {

    @PostConstruct
    public void init() {
      System.out.println("PrototypeBean.init");
    }

    @PreDestroy
    public void destroy() {
      System.out.println("PrototypeBean.destroy");
    }
  }

}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.47.38.png)

각각의 `find` 뒤에 `init`이 호출되었다. 1, 2번은 서로 다른 참조값을 가지고 있고 `destroy`는 호출되지 않았다. 생성하고 초기화 한 뒤엔 그냥 넘겨버리고 관리를 하지 않는 것이다.

```java
public class PrototypeTest {

  @Test
  void prototypeBeanFind() {
    ...
    
    prototypeBean1.destroy();
    prototypeBean2.destroy();
  }
}
```

꼭 사용해야 한다면 직접 `destroy`를 호출해야 한다.

## 싱글턴 빈과 함께 사용 시 문제점

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.52.15.png)

1. 클라이언트 A가 스프링 컨테이너에 프로토타입 빈을 요청한다.
2. 스프링 컨테이너는 프로토타입 빈을 새로 생성해서 반환한다. 해당 빈의 count 필드 값은 0이다.
3. 클라이언트가 조회한 프로토타입 빈에 `addCount()`를 호출하면서 +1을 한다.
4. 프로토타입 `x01`의 count가 1이 된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.52.22.png)

1. 클라이언트 B가 스프링 컨테이너에 프로토타입 빈을 요청한다.
2. 스프링 컨테이너가 프로토타입 빈을 새로 생성해서 반환한다. count는 0이다.
3. 클라이언트가 조회한 프로토타입 빈에 `addCount()`를 호출해서 +1 한다.
4. 프로토타입 `x02`의 count가 1이 된다.

{% tabs %} {% tab title="SingletonWithPrototypeTest1.java" %}

```java
public class SingletonWithPrototypeTest1 {

  @Test
  void prototypeFind() {
  AnnotationConfigApplicationContext ac = 
      new AnnotationConfigApplicationContext(PrototypeBean.class);

    PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
    prototypeBean1.addCount();
    assertThat(prototypeBean1.getCount()).isEqualTo(1);

    PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
    prototypeBean2.addCount();
    assertThat(prototypeBean2.getCount()).isEqualTo(1);
  }

  @Scope("prototype")
  static class PrototypeBean {

    private int count = 0;

    public void addCount() {
      count++;
    }

    public int getCount() {
      return count;
    }

    @PostConstruct
    public void init() {
      System.out.println("PrototypeBean.init: " + this);
    }

    @PreDestroy
    public void destroy() {
      System.out.println("PrototypeBean.destroy" + this);
    }
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%2010.23.57.png)

각각의 프로토타입이 다른 인스턴스를 가지고 있다. 여기까진 당연한 결과다.

---

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.52.32.png)

이번에는 `clientBean`이라는 싱글턴 빈이 의존 관계 주입으로 프로토타입 빈을 주입받아보자. `clientBean`은 싱글턴이기 때문에 스프링 컨테이너 생성 시점에 생성과 의존 관계 주입이 발생한다.

1. clientBean은 의존 관계 자동 주입으로 프로토타입 빈을 요청한다.
2. 스프링 컨테이너는 프로토타입 빈을 생성해 clientBean에 반환한다. 프로토타입 빈의 count 필드는 0이다.
3. clientBean이 프로토타입 빈을 내부 필드에 보관한다. 즉, 참조값을 보관하고 해당 빈을 관리한다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.52.41.png)

4. 클라이언트 A가 clientBean을 스프링 컨테이너에 요청한다. 싱글턴이므로 항상 같은 clientBean이 반환된다.
5. 클라이언트 A가 clientBean.logic()을 호출한다.
6. 호출된 clientBean은 프로토타입 빈의 addCount()를 호출해서 증가시키고, count는 1이 된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-14%20오전%209.52.49.png)

7. 클라이언트 B가 clientBean을 스프링 컨테이너에 요청하고 싱글턴이므로 같은 clientBean을 받는다.
    - clientBean이 내부에 가지고 있는 프로토타입 빈은 이미 과거에 주입이 끝난 빈이다.
    - 주입 시점에 스프링 컨테이너에 요청할 때마다 프로토타입이 생성되는 것이지 사용할 때마다 생성되는 것이 아니다.
8. 클라이언트 Brㅏ clinetBean.logic()을 호출한다.
9. clientBean은 addCount()를 호출해서 count를 증가시키고 count는 2가 된다.

{% tabs %} {% tab title=".java" %}

```java
public class SingletonWithPrototypeTest1 {
  @Test
  void singletonClientUsePrototype() {
    // ClientBean, PrototypeBean 둘 다 컴포넌트 스캔을 해줘야 빈으로 등록된다.
    ApplicationContext ac =
        new AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);

    ClientBean clientBean1 = ac.getBean(ClientBean.class);
    int count1 = clientBean1.logic();
    assertThat(count1).isEqualTo(1);

    ClientBean clientBean2 = ac.getBean(ClientBean.class);
    int count2 = clientBean2.logic();
    assertThat(count2).isEqualTo(2);
  }

  @Scope("singleton")
  static class ClientBean {
     // 생성 시점에 이미 주입되어 계속 같은 걸 쓰게 된다.
    private final PrototypeBean prototypeBean;

    @Autowired
    public ClientBean(PrototypeBean prototypeBean) {
      this.prototypeBean = prototypeBean;
    }

    public int logic() {
      prototypeBean.addCount();
      return prototypeBean.getCount();
    }
  }

  @Scope("prototype")
  static class PrototypeBean {

    private int count = 0;

    public void addCount() {
      count++;
    }

    public int getCount() {
      return count;
    }

    @PostConstruct
    public void init() {
      System.out.println("PrototypeBean.init: " + this);
    }

    @PreDestroy
    public void destroy() {
      System.out.println("PrototypeBean.destroy" + this);
    }
  }
}
```

{% endtab %} {% endtabs %}

테스트가 정상적으로 통과되었다.

`ClientBean`은 생성 시점에만 `PrototypeBean`을 주입받는다. 즉, 싱글턴 빈은 생성 시점에만 의존 관계 주입을 받기 때문에 프로토타입 빈 자체는 새로 생성되어도 싱글톤 빈과 라이프 사이클이 함께 유지된다.

만약 `logic()`을 호출 할 때마다 새로운 프로토타입을 주입받고 싶다면 어떻게 해야 할까? 다음 장에서 살펴보자.

---

```text
clientA -> prototypeBean@x01
clientB -> prototypeBean@x02
```

참고로 여러 빈이 같은 프로토타입 빈을 주입 받으면 주입 받는 시점에 각각 새로운 프로토타입 빈을 생성한다.
