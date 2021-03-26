# 관심사의 분리

연극 역할을 누가 할지는 배우가 아니라 기획자 결정한다. 배우가 캐스팅까지 담당한다면 너무 다양한 책임을 가진 것이다.

- 배우는 본인 배역을 잘 수행하는 것에만 집중한다.
- 배우는 어떤 상대 배우가 오더라도 똑같이 공연할 수 있다.
- 공연을 구성하고, 배우를 섭외하고, 역할을 지정하는 책임은 기획자가 한다.
- 기획자를 만들어 배우와 기획자의 책임을 확실히 분리해야 한다.

## AppConfig

애플리케이션의 전체 동작 방식을 구성하기 위해, `구현 객체를 생성하고 연결하는 책임`을 가진 별도의 설정 클래스를 만든다.

`AppConfig`는 애플리케이션의 실제 동작에 필요한 구현 객체를 생성한다.

- MemberServiceImpl
- MemoryMemberRepository
- OrderServiceImpl
- FixDiscountPolicy

생성한 객체 인스턴스의 참조(레퍼런스)를 생성자를 통해 주입(연결)한다.

- MemberServiceImpl → MemoryMemberRepository
- OrderServiceImpl → MemoryMemberRepository, FixDiscountPolicy

{% tabs %} {% tab title="AppConfig.java" %}

```java
public class AppConfig {

  public MemberService memberService() {
    return new MemberServiceImpl(new MemoryMemberRepository());
  }

  public OrderService orderService() {
    return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());
  }

}
```

{% endtab %}{% tab title="MemberServiceImpl.java" %}

```java
public class MemberServiceImpl implements MemberService {

  private final MemberRepository memberRepository;

  public MemberServiceImpl(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }
}
```

{% endtab %}{% tab title="OrderServiceImpl.java" %}

```java
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;

  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

{% endtab %} {% endtabs %}

이제 `MemberServiceImpl`은 `MemoryMemberRepository`를 의존하지 않는다. 단지 `MemberRepository` 인터페이스만 의존한다.

`MemberServiceImpl`은 생성자를 통해 어떤 구현 객체가 주입될 지 알 수 없다. 오직 `AppConfig`라는 외부에 의해서
결정된다. `MemberServiceImpl`은 이제 의존 관계에 대한 고민을 외부에 맡기고 실행에만 집중하면 된다.

### 클래스 다이어그램

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-09%20오후%202.14.34.png)

`MemberServiceImpl`이 `MemberService`를 구현하고 `MemberRepository`에 의존하는 건 동일하다. 대신 `AppConfig`
가 `MemoryMemberRepository`를 생성하고 연결해준다.

- DIP가 지켜진다.
    - `MemberServiceImpl`은 `MemberRepository`라는 추상에만 의존하면 된다.
- 관심사의 분리가 이루어졌다.
    - 객체를 생성하고 연결하는 역할과 실행하는 역할이 명확이 분리되었다.

### 회원 객체 인스턴스 다이어그램

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-09%20오후%202.19.52.png)

`AppConfig` 객체는 `MemoryMemberRepository` 객체를 생성하고 그 참조값을 `MemberServiceImpl`을 생성하는 과정에서 생성자로 전달한다.

클라이언트인 `MemberServiceImpl` 입장에서 보면 의존 관계를 마치 외부에서 주입해주는 것 같다고 해서 DI(Dependency Injection) 즉, 의존 관계
주입이라고 한다.

{% tabs %} {% tab title="OrderServiceImpl.java" %}

```java
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;

  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

{% endtab %} {% endtabs %}

설계를 변경한 후로는 `OrderServiceImpl`이 `FixDiscountPolicy`를 의존하지 않는다. `DiscountPolicy` 인터페이스만 의존할 뿐이다.

`OrderServiceImpl` 입장에선 생성자를 통해 어떤 구현 객체를 주입받을 지 알 수 없다. 오직 `AppConfig`에서 결정한다. `OrderServiceImpl`은
실행에만 집중하면 된다.

{% tabs %} {% tab title="MemberApp.java" %}

```java
public class MemberApp {

  public static void main(String[] args) {
    AppConfig appConfig = new AppConfig();
    MemberService memberService = appConfig.memberService();

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
    AppConfig appConfig = new AppConfig();
    MemberService memberService = appConfig.memberService();
    OrderService orderService = appConfig.orderService();

    Long memberId = 1L;
    Member member = new Member(memberId, "memberA", Grade.VIP);
    memberService.join(member);

    Order order = orderService.createOrder(memberId, "itemA", 10000);

    System.out.println("order = " + order);
    System.out.println("order.calculatePrice() = " + order.calculatePrice());
  }

}

```

{% endtab %} {% tab title="MemberServiceTest.java" %}

```java
public class MemberServiceTest {

  MemberService memberService;

  @BeforeEach
  void beforeEach() {
    AppConfig appConfig = new AppConfig();
    memberService = appConfig.memberService();
  }
}
```

{% endtab %} {% tab title="OrderServiceTest.java" %}

```java
public class OrderServiceTest {

  MemberService memberService;
  OrderService orderService;

  @BeforeEach
  void beforeEach() {
    AppConfig appConfig = new AppConfig();
    memberService = appConfig.memberService();
    orderService = appConfig.orderService();
  }
}

```

{% endtab %} {% endtabs %}

메인 메서드와 테스트 코드도 `AppConfig`에서 받아오도록 수정한다.

## 정리

- `AppConfig`를 통해 관심사를 확실하게 분리했다.
  - 배역에 맞는 배우를 선택하는 공연 기획자처럼 애플리케이션이 어떻게 동작할지 전체 구성을 책임진다.
- 각 클래스들은 기능을 실행하는 책임만 지면 된다.