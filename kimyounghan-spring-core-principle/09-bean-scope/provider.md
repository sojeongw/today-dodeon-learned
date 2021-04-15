# Provider

싱글턴 빈과 프로토타입 빈을 함께 사용할 때, 사용할 때마다 새로운 프로토타입 빈을 생성하는 방법을 알아보자.

## 사용할 때마다 스프링 컨테이너에 새로 요청

{% tabs %} {% tab title="SingletonWithPrototypeTest1.java" %}

```java
public class SingletonWithPrototypeTest1 {

  @Test
  void providerTest() {
    AnnotationConfigApplicationContext ac =
        new AnnotationConfigApplicationContext(PrototypeBean.class);

    PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
    prototypeBean1.addCount();
    assertThat(prototypeBean1.getCount()).isEqualTo(1);

    PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
    prototypeBean2.addCount();
    assertThat(prototypeBean2.getCount()).isEqualTo(1);
  }

  @Scope("singleton")
  static class ClientBean {

    @Autowired
    private ApplicationContext ac;

    // 사용할 때마다 스프링 컨테이너에 새로 요청한다.
    public int logic() {
      PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
      prototypeBean.addCount();
      return prototypeBean.getCount();
    }
  }

  ...
}
```

{% endtab %} {% endtabs %}

이렇게 의존 관계를 외부에서 주입받지 않고 직접 찾는 것을 Dependency Lookup(DL), 의존 관계 조회(탐색)이라고 한다.

그런데 이렇게 스프링 애플리케이션 컨텍스트 자체를 주입받게 되면 스프링 컨테이너에 종속적인 코드가 되고 단위 테스트가 어려워진다.

지금 필요한 기능은 지정한 프로토타입 빈을 컨테이너에서 대신 찾아주는 DL 정도의 기능만 있으면 된다.

## ObjectFactory, ObjectProvider

`ObjectProvider`는 지정한 빈을 컨테이너에서 대신 찾아주는 DL을 제공한다. 과거에는 `ObjectFactory`를 썼으나 여기에 편의 기능을
추가해 `ObjectProvider`가 만들어졌다.

{% tabs %} {% tab title="SingletonWithPrototypeTest1.java" %}

```java
public class SingletonWithPrototypeTest1 {

  ...

  @Scope("singleton")
  static class ClientBean {

    @Autowired
    private ObjectProvider<PrototypeBean> prototypeBeanProvider;

    public int logic() {
      // 프로토타입 빈을 대신 찾아준다.
      PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
      prototypeBean.addCount();
      return prototypeBean.getCount();
    }
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-15%20오전%2010.35.39.png)

프로토타입 빈이 각각 다른 참조값으로 생성되었다.

`ObjectProvider`는 `getObject()`를 호출하면 스프링 컨테이너를 통해 해당 빈을 찾아 반환한다. 즉, 딱 필요한 정도의 DL 기능을 제공한다. 프로토타입 빈을
전용으로 관리한다기 보다는 찾아주는 과정을 간단하게 도와주는 존재라고 생각하면 된다.

스프링이 제공하는 기능을 사용하지만 기능이 단순하기 때문에 단위 테스트를 만들거나 mock 코드를 만들기 쉬워진다.

### ObjectFactory

- 기능이 단순하다.
    - getObject()만 제공한다.
- 별도의 라이브러리가 필요없다.
- 스프링에 의존적이다.

### ObjectProvider

- ObjectFactory를 상속한다.
- 상속, 옵션, 스트림 처리 등 편의 기능이 많다.
- 별도의 라이브러리가 필요없다.
- 스프링에 의존한다.

## JSR-330 Provider

`javax.inject.Provider`라는 JSR-330 자바 표준을 사용하는 방법이다.
gradle에 `implementation('javax.inject:javax.inject:1')`을 추가해줘야 하는 단점이 있다.

{% tabs %} {% tab title="SingletonWithPrototypeTest1.java" %}

```java
public class SingletonWithPrototypeTest1 {

  ...

  @Scope("singleton")
  static class ClientBean {

    // javax.inject의 Provider를 가져와야 한다.
    @Autowired
    private Provider<PrototypeBean> prototypeBeanProvider;

    public int logic() {
      PrototypeBean prototypeBean = prototypeBeanProvider.get();
      prototypeBean.addCount();
      return prototypeBean.getCount();
    }
  }
}
```

{% endtab %} {% endtabs %}

`provider.get()`을 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아 반환하는 딱 DL만큼의 기능을 제공한다. 

- 자바 표준 
- 기능이 단순하다.
    - get() 메서드 하나로 동작한다.
    - 단위 테스트를 만들거나 mock 코드를 만들기 훨씬 쉬워진다.
- 별도의 라이브러리가 필요하다.
- 자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용할 수 있다.

## 프로토타입은 언제 사용할까

매번 사용할 때마다 의존 관계 주입이 완료된 새로운 객체가 필요할 때 사용한다. 하지만 실무에서는 싱글턴 빈으로 대부분의 문제가 해결되기 때문에 프로토타입 빈을 직접 사용하는 일은 드물다.

## Provider의 용도

`ObjectProvider`나 `JSR330 Provider` 등은 프토토타입이 꼭 아니어도 DL이 필요한 경우라면 언제든 사용할 수 있다.

- retrieving multiple instances.
- lazy or optional retrieval of an instance.
- breaking circular dependencies.
    - A가 B를 의존하고 B가 A를 필요로 할 때 사용하면 a가 b를 사용할때 b는 당장 a를 사용하지 않아도 되면서 순환 참조가 해결된다.
- abstracting scope so you can look up an instance in a smaller scope from an instance in a containing scope.

Provider 외에도 메서드에 `@Lookup`을 사용하는 방식이 있으나 이전 방법들로 충분하고 고려해야할 내용도 많아서 생략한다.

## ObjectProvider vs JSR-330 Provider

`ObjectProvider`는 스프링 외에는 추가가 필요 없어서 편리하다. 만약 스프링이 아닌 컨테이너를 사용해야 한다면 `JSR-330 Provider`을 사용한다.

외에도 스프링 기능과 자바 표준이 겹칠 때가 있는데 스프링이 대부분 더 다양하고 편리한 기능을 제공하기 때문에 특별히 다른 컨테이너를 사용할 일이 없다면 스프링의 기능을 사용하면 된다.
