# 스프링으로 전환하기

{% tabs %} {% tab title="AppConfig.java" %}

```java
// 설정 정보라는 의미로 붙여준다.
@Configuration
public class AppConfig {

  // 스프링 컨테이너에 빈으로 등록한다.
  @Bean
  public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
  }

  @Bean
  public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
  }

  public OrderService orderService() {
    return new OrderServiceImpl(memberRepository(), discountPolicy());
  }

  @Bean
  public DiscountPolicy discountPolicy() {
    return new RateDiscountPolicy();
  }

}
```

{% endtab %} {% tab title="MemberApp.java" %}

```java
public class MemberApp {

  public static void main(String[] args) {
//    AppConfig appConfig = new AppConfig();
//    MemberService memberService = appConfig.memberService();

    // AppConfig에 있는 환경 설정 정보를 가지고 해당 객체를 빈으로 스프링 컨테이너에 다 넣어준다.
    ApplicationContext applicationContext =
        new AnnotationConfigApplicationContext(AppConfig.class);
    // 해당 빈을 가져온다.
    MemberService memberService =
        applicationContext.getBean("memberService", MemberService.class);

    // 빈은 AppConfig에서 설정한 구현체를 끌고 온다.
    Member member = new Member(1L, "memberA", Grade.VIP);
    memberService.join(member);

    Member findMember = memberService.findMember(1L);
    System.out.println("new member = " + member.getName());
    System.out.println("findMember = " + findMember.getName());
  }
}
```

{% endtab %} {% tab title="OrderApp.java" %}

```java
public class OrderApp {

  public static void main(String[] args) {
//    AppConfig appConfig = new AppConfig();
//    MemberService memberService = appConfig.memberService();
//    OrderService orderService = appConfig.orderService();

    ApplicationContext applicationContext =
        new AnnotationConfigApplicationContext(AppConfig.class);
    OrderService orderService =
        applicationContext.getBean("orderService", OrderService.class);
    MemberService memberService =
        applicationContext.getBean("memberService", MemberService.class);

    Long memberId = 1L;
    Member member = new Member(memberId, "memberA", Grade.VIP);
    memberService.join(member);

    Order order = orderService.createOrder(memberId, "itemA", 20000);

    System.out.println("order = " + order);
    System.out.println("order.calculatePrice() = " + order.calculatePrice());
  }

}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-10%20오후%2012.55.32.png)

로그를 보면 결과를 출력하기 전에 빈을 등록하는 과정이 있다.

## 스프링 컨테이너

`ApplicationContext`를 스프링 컨테이너라고 한다. 기존에는 개발자가 `AppConfig`로 직접 객체를 생성하고 DI를 했지만 이제는 스프링 컨테이너를 사용한다.

스프링 컨테이너는 `@Configuration`이 붙은 `AppConfig`를 설정(구성) 정보로 사용한다. `@Bean`이 적힌 메서드를 모두 호출한 뒤 반한된 객체를 스프링 컨테이너에 등록한다. 이렇게 스프링 컨테이너에 등록된 객체를 스프링 빈이라고 한다.

스프링 빈은 `@Bean`이 붙은 메서드 명을 스프링 빈의 이름으로 사용한다. 예제에서는 `memberService`, `orderService`가 된다.

이전에는 개발자가 `AppConfig`를 사용해 필요한 객체를 직접 조회했지만, 이제는 스프링 컨테이너를 통해 필요한 스프링 빈을 찾아야 한다. `applicationContext.getBean()` 메서드로 조회할 수 있다.

지금까지는 개발자가 직접 자바 코드를 모든 것을 했다면, 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아 사용하도록 변경되었다.