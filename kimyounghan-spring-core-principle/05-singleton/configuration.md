# @Configuration과 싱글턴

{% tabs %} {% tab title="AppConfig.java" %}

```java

@Configuration
public class AppConfig {

  @Bean
  public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
  }

  @Bean
  public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
  }

  @Bean
  public OrderService orderService() {
    return new OrderServiceImpl(memberRepository(), discountPolicy());
  }

  @Bean
  public DiscountPolicy discountPolicy() {
    return new RateDiscountPolicy();
  }

}

```

{% endtab %} {% endtabs %}

`memberService`와 `orderService` 둘 다 `new MemoryMemberRepository`를 호출하는 `memberRepository`를 호출한다. 각각
다른 `MemoryMemberRepository`가 생성되면서 싱글턴이 깨지는 것처럼 보인다.

{% tabs %} {% tab title="MemberServiceImpl.java" %}

```java
public class MemberServiceImpl implements MemberService {

  private final MemberRepository memberRepository;

  // 테스트 용도
  public MemberRepository getMemberRepository() {
    return memberRepository;
  }
}
```

{% endtab %} {% tab title="OrderServiceImpl.java" %}

```java
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;

  // 테스트 용도
  public MemberRepository getMemberRepository() {
    return memberRepository;
  }
}
```

{% endtab %} {% tab title="ConfigurationSingletonTest.java" %}

```java
public class ConfigurationSingletonTest {

  @Test
  void configurationTest() {
    AnnotationConfigApplicationContext ac =
        new AnnotationConfigApplicationContext(AppConfig.class);

    // 테스트 용도 메서드를 불러오기 위해 구체 클래스로 불러오지만 이런 방식은 원래 지양해야 한다.
    MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
    OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
    // 저장소 자체도 확인해보자.
    MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

    MemberRepository memberRepository1 = memberService.getMemberRepository();
    MemberRepository memberRepository2 = orderService.getMemberRepository();

    System.out.println("memberRepository1 = " + memberRepository1);
    System.out.println("memberRepository2 = " + memberRepository2);
    System.out.println("memberRepository = " + memberRepository);

    assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
    assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
  }

}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%201.42.18.png)

테스트 해보면 같은 참조값이 나온다. `memberRepository` 인스턴스가 모두 같은 인스턴스로 공유되고 있는 것이다.

혹시 메서드가 두 번 호출이 안 되는게 아닐까? 로그를 찍어보자.

{% tabs %} {% tab title="AppConfig.java" %}

```java

@Configuration
public class AppConfig {

  @Bean
  public MemberService memberService() {
    System.out.println("called AppConfig.memberService");
    return new MemberServiceImpl(memberRepository());
  }

  @Bean
  public MemberRepository memberRepository() {
    System.out.println("called AppConfig.memberRepository");
    return new MemoryMemberRepository();
  }

  @Bean
  public OrderService orderService() {
    System.out.println("called AppConfig.orderService");
    return new OrderServiceImpl(memberRepository(), discountPolicy());
  }

  @Bean
  public DiscountPolicy discountPolicy() {
    return new RateDiscountPolicy();
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%201.56.26.png)

스프링 컨테이너는 각각의 `@Bean`을 호출해서 스프링 빈을 생성한다. 따라서 `@Bean`이 붙어있는  `memberRepository` 1번에, `memberService`
와 `orderService`에서 각각 호출하는 횟수 2번까지 총 3번이 찍혀야할 것 같다.

하지만 결과를 보면 신기하게도 한 번씩만 찍혔다.

## @Configuration과 바이트 코드 조작의 마법

스프링 컨테이너는 싱글턴 객체를 생성하고 관리하는 싱글턴 레지스트리다. 따라서 스프링 빈이 싱글턴이 되도록 보장해야 한다.

{% tabs %} {% tab title="ConfigurationSingletonTest.java" %}

```java
public class ConfigurationSingletonTest {

  @Test
  void configurationDeep() {
    AnnotationConfigApplicationContext ac =
        new AnnotationConfigApplicationContext(AppConfig.class);

    // config로 넘긴 클래스도 스프링 빈으로 등록이 된다.
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean.getClass() = " + bean.getClass());
  }

}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%202.08.03.png)

스프링 빈 `AppConfig`의 클래스 정보를 출력하면 위와 같다. 순수한 클래스라면 `class hello.core.AppConfig`로 출력되어야 한다.

스프링이 `CGLIB`이라는 바이트 코드 조작 라이브러리로 `AppConfig` 클래스를 상속한 임의의 다른 클래스를 만들어 스프링 빈으로 등록한 것이다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%202.11.05.png)

이 조작해서 만든 다른 클래스가 싱글턴이 되도록 보장한다.

```java
@Bean
public MemberRepository memberRepository() {
    if(memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면){
      return 스프링 컨테이너에서 찾아서 반환;
    } else { 
      //스프링 컨테이너에 없으면
      기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
      
      return 반환;
    }
}
```

바이트 코드를 저장하는 과정은 대략 이렇게 될 것이다. 

`@Bean`이 붙은 메서드마다 스프링 빈이 있는지 확인한 뒤 있으면 그 빈을 반환하고 없으면 새로 생성해서 빈으로 등록하고 반환하는 코드가 동적으로 생성된다. 이 덕분에 싱글턴이 보장되는 것이다.

```java
public class ConfigurationSingletonTest {

  @Test
  void configurationDeep() {
    ...
    AppConfig bean = ac.getBean(AppConfig.class);
    System.out.println("bean.getClass() = " + bean.getClass());
  }

}
```

참고로 `AppConfig@CGLIB`은 `AppConfig`의 자식 타입이므로 이전의 테스트 코드에서 `AppConfig`로 조회할 수 있었던 것이다.

### @Configuration을 적용하지 않는다면?

`@Configuration`을 붙이면 바이트 코드를 조작하는 CGLIB 기술로 싱글턴을 보장한다. 하지만 만약 `@Bean`만 적용한다면 어떻게 될까?

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%202.20.01.png)

CGLIB 기술 없이 순수한 `AppConfig`로 스프링 빈에 등록된 것을 알 수 있다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%202.20.55.png)

`memberRepository`가 3번 호출되고 각각이 다른 인스턴스로 생성되었다.

`@Bean`만 사용해도 스프링 빈으로 등록은 되지만, 싱글턴을 보장하지 않는다. 이전에는 CGLIB이 스프링 컨테이너에 있는지 확인하고 컨테이너에서 찾아 반환해줬다. 하지만 이제는  `memberService`와 `orderService`에서 의존하는 `memberRepository`는 더 이상 스프링 컨테이너가 관리하는 객체가 아니기 때문에 사실상 `new MemoryMemberRepository()`로 가져오는 것과 다름 없다.

따라서 스프링 설정 정보는 항상 `@Configuration`을 사용하자.