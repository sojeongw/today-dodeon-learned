# 주문과 할인 도메인 개발
## 구현

{% tabs %} {% tab title="DiscountPolicy.java" %}

```java
public interface DiscountPolicy {

  /**
   *
   * @return 할인 대상 금액
   */
  int discount(Member member, int itemPrice);

}
```

{% endtab %} {% tab title="FixDiscountPolicy.java" %}

```java
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

{% endtab %} {% tab title="Order.java" %}

```java
public class Order {

  private Long memberId;
  private String itemName;
  private int itemPrice;
  private int discountPrice;

  public Order(Long memberId, String itemName, int itemPrice, int discountPrice) {
    this.memberId = memberId;
    this.itemName = itemName;
    this.itemPrice = itemPrice;
    this.discountPrice = discountPrice;
  }

  public int calculatePrice() {
    return itemPrice - discountPrice;
  }

  @Override
  public String toString() {
    return "Order{" +
        "memberId=" + memberId +
        ", itemName='" + itemName + '\'' +
        ", itemPrice=" + itemPrice +
        ", discountPrice=" + discountPrice +
        '}';
  }
}
```

{% endtab %} {% tab title="OrderService.java" %}

```java
public interface OrderService {

  Order createOrder(Long memberId, String itemName, int itemPrice);
}

```

{% endtab %} {% tab title="OrderServiceImpl.java" %}

```java
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository = new MemoryMemberRepository();
  private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

  @Override
  public Order createOrder(Long memberId, String itemName, int itemPrice) {
    Member member = memberRepository.findById(memberId);
    // 난 모르겠고 할인 정책은 할인 정책에게 넘긴다.
    int discountPrice = discountPolicy.discount(member, itemPrice);

    return new Order(memberId, itemName, itemPrice, discountPrice);
  }
}
```

{% endtab %} {% endtabs %}

### 테스트

{% tabs %} {% tab title="OrderApp.java" %}

```java
public class OrderApp {

  public static void main(String[] args) {
    MemberService memberService = new MemberServiceImpl();
    OrderService orderService = new OrderServiceImpl();

    Long memberId = 1L;
    Member member = new Member(memberId, "memberA", Grade.VIP);
    memberService.join(member);

    Order order = orderService.createOrder(memberId, "itemA", 10000);

    System.out.println("order = " + order);
    System.out.println("order.calculatePrice() = " + order.calculatePrice());
  }

}
```

{% endtab %} {% endtabs %}

```text
order = Order{memberId=1, itemName='itemA', itemPrice=10000, discountPrice=1000}
order.calculatePrice() = 9000
```

{% tabs %} {% tab title="OrderServiceTest.java" %}

```java
public class OrderServiceTest {

  MemberService memberService = new MemberServiceImpl();
  OrderService orderService = new OrderServiceImpl();

  @Test
  void createOrder() {
    // 원시 타입으로 하면 null을 넣을 수 없어서 래퍼 타입으로 넣는다.
    Long memberId = 1L;
    Member member = new Member(memberId, "memberA", Grade.VIP);
    memberService.join(member);

    Order order = orderService.createOrder(memberId, "itemA", 10000);

    Assertions.assertThat(order.getDiscountPrice()).isEqualTo(1000);
  }

}
```

{% endtab %} {% endtabs %}