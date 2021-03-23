# 스프링 핵심 원리 이해 - 객체 지향 원리 적용
## 새로운 할인 정책

```text
악덕 기획자: 서비스 오픈 직전에 할인 정책을 지금처럼 고정 금액 할인이 아니라 좀 더 합리적인 주문 금액 당 할인하는 정률 % 할인으로 변경하고 싶어요. 
예를 들어서 기존 정책은 VIP가 10000원을 주문하든 20000원을 주문하든 항상 1000원을 할인했는데, 이번에 새로 나온 정책은 10%로 지정해두면
고객이 10000원 주문시 1000원을 할인해주고, 20000원 주문시에 2000원을 할인해주는 거에요!

순진 개발자: 제가 처음부터 고정 금액 할인은 아니라고 했잖아요.

악덕 기획자: 애자일 소프트웨어 개발 선언 몰라요? “계획을 따르기보다 변화에 대응하기를”

순진 개발자: ... (하지만 난 유연한 설계가 가능하도록 객체지향 설계 원칙을 준수했지 후후)
```

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-09%20오전%209.55.07.png)

객체 지향적으로 설계했다면 `RateDiscountPolicy`로 바꾸기만 하면 될 것이다. 확인해보자.

{% tabs %} {% tab title="RateDiscountPolicy.java" %}

```java
public class RateDiscountPolicy implements DiscountPolicy{

  private int discountPercent = 10;

  @Override
  public int discount(Member member, int itemPrice) {
    if(member.getGrade() == Grade.VIP) {
      return itemPrice * discountPercent / 100;
    } else {
      return 0;
    }
  }
}

```

{% endtab %} {% tab title="RateDiscountPolicyTest.java" %}

```java
class RateDiscountPolicyTest {

  RateDiscountPolicy discountPolicy = new RateDiscountPolicy();

  @Test
  @DisplayName("VIP는 10% 할인이 적용되어야 한다")
  void vip_discount() {
    // given
    Member member = new Member(1L, "memberVIP", Grade.VIP);

    // when
    int discount = discountPolicy.discount(member, 10000);

    // then
    Assertions.assertThat(discount).isEqualTo(1000);
  }

  @Test
  @DisplayName("VIP가 아니면 할인이 적용되지 않아야 한다")
  void not_vip() {
    // given
    Member member = new Member(1L, "memberBasic", Grade.BASIC);

    // when
    int discount = discountPolicy.discount(member, 10000);

    // then
    Assertions.assertThat(discount).isEqualTo(0);
  }
}
```

{% endtab %} {% endtabs %}

## 새로운 할인 정책 적용과 문제점

{% tabs %} {% tab title="OrderServiceImpl.java" %}

```java
public class OrderServiceImpl implements OrderService{

  private final MemberRepository memberRepository = new MemoryMemberRepository();
//  private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
  private final DiscountPolicy discountPolicy = new RateDiscountPolicy();

  @Override
  public Order createOrder(Long memberId, String itemName, int itemPrice) {
    Member member = memberRepository.findById(memberId);
    int discountPrice = discountPolicy.discount(member, itemPrice);

    return new Order(memberId, itemName, itemPrice, discountPrice);
  }
}
```

{% endtab %} {% endtabs %}

할인 정책을 변경하려면 클라이언트인 `OrderServiceImpl` 코드를 `FixDiscountPolicy`에서 `RateDiscountPolicy`로 고쳐야 한다.

우리는 역할과 구현을 충실하게 분리했다. 다형성도 활용하고 인터페이스와 구현 객체를 분리했다. OCP, DIP 같은 객체 지향 설계 원칙은 충실히 준수한 것 같지만 사실은 아니다.

### DIP

```text
주문 서비스 클라이언트 OrderServiceImpl은 DiscountPolicy 인터페이스에 의존하면서 DIP를 지킨 것 같은데?!
```

클래스 의존 관계를 분석해보면 추상(인터페이스) `DiscountPolicy` 뿐만 아니라 구체(구현) `FixDiscountPolicy`, `RateDiscountPolicy` 클래스에도 의존하고 있다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-09%20오전%2010.10.52.png)

지금까지는 단순히 `DiscountPolicy` 인터페이스에만 의존한다고 생각했다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-09%20오전%2010.10.58.png)

하지만 잘 보면 클라이언트인 `OrderServiceImpl`이 `DiscountPolicy`뿐만 아니라 구체 클래스인 `FixDiscountPolicy`도 의존하고 있다. DIP를 위반하는 것이다.

### OCP

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-09%20오전%2010.13.37.png)

`OrderServiceImpl`이 인터페이스만 보는 게 아니라 `FixDiscountPolicy`도 보고 있기 때문에 `RateDiscountPolicy`로 변경하는 순간 `OrderServiceImpl`의 소스 코드도 함꼐 변경해야 한다.

기능을 확장해서 변경하면 클라이언트 코드에 영향을 주므로 OCP를 위반한다.

기름차를 전기차로 변경했는데 라이센스를 다시 따야하는 상황이 온 것이다.

## 어떻게 해결할 수 있을까?

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-09%20오전%2010.18.18.png)

인터페이스에만 의존하도록 설계를 변경하자.

{% tabs %} {% tab title="OrderServiceImpl.java" %}

```java
public class OrderServiceImpl implements OrderService{

  private DiscountPolicy discountPolicy;

  ...
}

```

{% endtab %} {% endtabs %}

인터페이스에만 의존하도록 설계와 코드를 변경했다. 그런데 구현체가 없는 상태에서 코드를 실행하면 NPE가 발생한다.

이 문제를 해결하려면 누군가가 클라이언트인 `OrderServiceImpl`에 `DiscountPolicy`의 구현 객체를 대신 생성하고 주입해줘야 한다.