# 조회 빈이 2개 이상일 때

```java
@Autowired
private DiscountPolicy discountPolicy
```

`@Autowired`는 타입으로 조회하기 때문에 `ac.getBean(DiscountPolicy.class)`와 유사하게 동작한다.

그런데 이 코드는 타입으로 조회할 때 선택된 빈이 2개 이상이면 문제가 발생한다.

```java

@Component
public class FixDiscountPolicy implements DiscountPolicy {

}

@Component
public class RateDiscountPolicy implements DiscountPolicy {

}

public class Client {

  // 의존성 주입 실패
  @Autowired
  private DiscountPolicy discountPolicy;
}
```

```text
NoUniqueBeanDefinitionException: No qualifying bean of type
'hello.core.discount.DiscountPolicy' available: expected single matching bean
but found 2: fixDiscountPolicy,rateDiscountPolicy
```

이렇게 `DiscountPolicy`의 하위 타입 2개를 모두 스프링 빈으로 선언하면  `NoUniqueBeanDefinitionException` 오류가 발생한다.

의존성 주입하는 부분에서 `DiscountPolicy` 대신 하위 타입으로 지정할 수도 있지만 이런 방법은 DIP를 위배하고 유연성이 떨어진다. 게다가 이름만 다르고 완전히 똑같은
타입의 스프링 빈이 2개 있는 상황이라면 해결되지 않는다.

## @Autowired 필드 명 매칭

{% tabs %} {% tab title="Before" %}

```java
@Autowired
private DiscountPolicy discountPolicy;
```

{% endtab %} {% tab title="After" %}

```java
@Autowired
private DiscountPolicy rateDiscountPolicy;

```

{% endtab %} {% endtabs %}

필드 명을 `rateDiscountPolicy`로 지정해줬으므로 정상적으로 주입된다. 필드명 매칭은 먼저 타입 매칭을 시도한 뒤 그 결과에 여러 빈이 존재할 때 추가로 동작하는 기능이다.

1. 타입 매칭
2. 2개 이상일 경우 필드 명, 파라미터 명으로 빈 이름 매칭

## @Qualifier

```java

@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {

}

@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {

}
```

`@Qualifier`를 붙여 구분해준다. 주입할 떄 추가적인 방법을 제공하는 것이지 빈 이름을 변경하는 것은 아니다.

{% tabs %} {% tab title="생성자 자동 주입" %}

```java
public class OrderServiceImpl {

  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository,
      // 주입할 빈 지정
      @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

{% endtab %} {% tab title="수정자 자동 주입" %}

```java
public class OrderServiceImpl {

  @Autowired
  public DiscountPolicy setDiscountPolicy(
      // 주입할 빈 지정
      @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy
  ) {
    return discountPolicy;
  }
}
```

{% endtab %} {% tab title="직접 Bean 등록" %}

```java
public class AppConfig {

  // 직접 빈을 등록하는 경우는 이렇게 사용할 수 있다.
  @Bean
  @Qualifier("mainDiscountPolicy")
  public DiscountPolicy discountPolicy() {
    return new ...
  }
}
```

{% endtab %} {% endtabs %}

`@Qualifier`로 주입할 때 `@Qualifier("mainDiscountPolicy")`를 못찾으면 `mainDiscountPolicy`라는 이름의 스프링 빈을 추가적으로 찾는다. 하지만 경험상 `@Qualifier`는 `@Qualifier`를 찾는 용도로만 사용하는 게 명확하고 좋다.

1. @Qualifier끼리 매칭
2. 빈 이름 매칭
3. 없으면 `NoSuchBeanDefinitionException` 발생

## @Primary

우선 순위를 정하는 방법이다. `@Autowired`할 때 여러 빈이 매칭되면 `@Primary` 붙은 빈이 우선권을 가진다.

{% tabs %} {% tab title="@Primary 적용" %}

```java

@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {

}

@Component
public class FixDiscountPolicy implements DiscountPolicy {

}
```

{% endtab %} {% tab title="사용 코드" %}

```java
public class OrderServiceImpl {

  // 생성자
  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository,
      DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }

  // 수정자
  @Autowired
  public DiscountPolicy setDiscountPolicy(DiscountPolicy discountPolicy) {
    return discountPolicy;
  }
}
```

{% endtab %} {% endtabs %}

---

`@Primary`와 `@Qualifier` 중에 어떤 것을 선택하는 게 좋을까?

{% tabs %} {% tab title="생성자 자동 주입" %}

```java
public class OrderServiceImpl {

  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository,
      @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

{% endtab %} {% tab title="수정자 자동 주입" %}

```java
public class OrderServiceImpl {

  @Autowired
  public DiscountPolicy setDiscountPolicy(
      @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy
  ) {
    return discountPolicy;
  }
}
```

{% endtab %} {% endtabs %}

---

`@Qualifier`는 주입 받을 때 모든 코드에 애너테이션을 붙여줘야 하는 단점이 있다. 반면에 `@Primary`는 이렇게 붙일 필요가 없다.

메인 DB와 서브 DB가 존재할 때 메인에 `@Primary`를 적용해 `@Qualifier` 지정 없이 편하게 조회하고, 서브를 읽을 때는 `@Qualifier`를 이용해 명시적으로 가져오면 깔끔하게 사용할 수 있다.

스프링은 자동보다 **수동**이, 넓은 선택권 보다는 **좁은 선택권**이 우선 순위가 높다. `@Primary`는 기본값처럼 동작하지만 `@Qualifier`는 매우 상세하게 동작하는 기능이므로 `@Qualifier`가 우선권이 높다.