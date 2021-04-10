# 애너테이션 직접 만들기

`@Qualifier("mainDiscountPolicy")`로 쓰면 String이므로 컴파일 할 떄 타입 체크가 안된다.

{% tabs %} {% tab title="MainDiscountPolicy.java" %}

```java
// 애너테이션을 직접 만든다.
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER,
    ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {

}
```

{% endtab %} {% tab title="RateDiscountPolicy.java" %}

```java

@Component
// 직접 만든 애너테이션을 적용한다.
@MainDiscountPolicy
public class RateDiscountPolicy implements DiscountPolicy {

}
```

{% endtab %} {% tab title="OrderServiceImpl.java" %}

```java
public class OrderServiceImpl implements OrderService {

  // 생성자 자동 주입
  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository,
      @MainDiscountPolicy DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }

  // 수정자 자동 주입
  @Autowired
  public DiscountPolicy setDiscountPolicy(
      @MainDiscountPolicy DiscountPolicy discountPolicy
  ) {
    return discountPolicy;
  }
}
```

{% endtab %} {% endtabs %}

애너테이션에는 상속 개념이 없다. 이렇게 여러 애너테이션을 모아서 사용하는 기능은 스프링이 지원하는 기능이다. `@Qualifier`외에 다른 애너테이션도 함께 조합해서 쓸 수 있다.

스프링이 제공하는 기능을 뚜렷한 목적없이 무분별하게 재정의하면 유지보수할 때 혼란을 가중시킬 수 있다.