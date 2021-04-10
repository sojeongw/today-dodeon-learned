# 조회한 빈이 모두 필요할 때

정말 해당 타입의 스프링 빈이 다 필요한 경우도 있다. 할인 서비스를 제공할 때 클라이언트가 할인 종류를 선택하고 싶을 수 있다.

{% tabs %} {% tab title="AllBeanTest.java" %}

```java
public class AllBeanTest {

  @Test
  void findAllBean() {
    // 두 클래스를 스프링 빈으로 등록한다.
    ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class,
        DiscountService.class);
  }

  static class DiscountService {

    private final Map<String, DiscountPolicy> policyMap;
    private final List<DiscountPolicy> policyList;

    // 빈을 등록할 때 맵과 리스트를 주입받는다.
    @Autowired
    public DiscountService(
        Map<String, DiscountPolicy> policyMap,
        List<DiscountPolicy> policyList) {
      this.policyMap = policyMap;
      this.policyList = policyList;

      System.out.println("policyMap = " + policyMap);
      System.out.println("policyList = " + policyList);
    }
  }
}
```

{% endtab %} {% tab title="AutoAppConfig.java" %}

```java

@Configuration
@ComponentScan(
    excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = Configuration.class),
    basePackages = "hello.core"
)
public class AutoAppConfig {

}

```

{% endtab %} {% tab title="RateDiscountPolicy.java" %}

```java

@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {

  private int discountPercent = 10;

  @Override
  public int discount(Member member, int itemPrice) {
    if (member.getGrade() == Grade.VIP) {
      return itemPrice * discountPercent / 100;
    } else {
      return 0;
    }
  }
}
```

{% endtab %} {% tab title="FixDiscountPolicy.java" %}

```java

@Component
public class FixDiscountPolicy implements DiscountPolicy {

  private int discountFixAmount = 1000;

  @Override
  public int discount(Member member, int itemPrice) {
    if (member.getGrade() == Grade.VIP) {
      return discountFixAmount;
    } else {
      return 0;
    }
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/07/screenshot%202021-04-12%20오전%209.39.06.png)

`DiscountPolicy`에 속하는 `FixDiscountPolicy`와 `RateDiscountPolicy`가 모두 주입되었다.

`Map<String, DiscountPolicy>`은 키에 스프링 빈의 이름, 값으로 `DiscountPolicy` 타입으로 조회한 모든 스프링 빈이 담긴다.

`List<DiscountPolicy>`는 `DiscountPolicy` 타입으로 조회한 모든 스프링 빈을 담아준다.

만약 해당 타입의 스프링 빈이 없다면 빈 컬렉션을 주입한다.

{% tabs %} {% tab title="AllBeanTest.java" %}

```java
public class AllBeanTest {

  @Test
  void findAllBean() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class,
        DiscountService.class);

    DiscountService discountService = ac.getBean(DiscountService.class);
    Member member = new Member(1L, "userA", Grade.VIP);
    int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");

    assertThat(discountService).isInstanceOf(DiscountService.class);
    assertThat(discountPrice).isEqualTo(1000);

    int rateDiscountPrice = discountService.discount(member, 20000, "rateDiscountPolicy");
    assertThat(rateDiscountPrice).isEqualTo(2000);
  }

  static class DiscountService {
    ...

    // 넘겨준 정책이 discountCode가 되어 해당 정책을 찾아 계산한 뒤 리턴한다.
    public int discount(Member member, int price, String discountCode) {
      DiscountPolicy discountPolicy = policyMap.get(discountCode);
      return discountPolicy.discount(member, price);
    }
  }
}
```

{% endtab %} {% endtabs %}

클라이언트에서 할인 정책을 선택하면 그 정책에 따라 할인 금액을 계산한다.