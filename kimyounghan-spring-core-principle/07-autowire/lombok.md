# 롬복과 최신 트렌드

생성자를 만들고 주입 받은 값을 대입하는 코드를 만드는 방식은 번거롭다. 필드 주입처럼 편리하게 사용할 수 있는 방법을 소개한다.

{% tabs %} {% tab title="Before.java" %}

```java

@Component
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;

  @Autowired  // 생략 가능
  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

{% endtab %} {% tab title="After.java" %}

```java

@Component
// 롬복이 final이 붙은 필드를 모아 생성자를 자동으로 만들어준다.
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;
}

```

{% endtab %} {% endtabs %}

롬복이 자바의 애너테이션 프로세서라는 기능을 이용해 컴파일 시점에 생성자 코드를 자동으로 생성해준다.

```java
public class OrderServiceImpl implements OrderService {

  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

실제 `class` 파일을 열어보면 위와 같은 코드가 추가되어 있다.

최근에는 생성자를 딱 1개 두고 `@Autowired`를 생략하는 방법을 주로 사용한다. 여기에 롬복 라이브러리의 `@RequiredArgsConstructor`를 함께 사용하면 기능은 다 제공하면서 코드는 깔끔하게 사용할 수 있다.