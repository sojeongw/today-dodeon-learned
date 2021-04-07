# 다양한 의존 관계 주입 방법

## 생성자 주입

```java

@Component
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;

  // 생성자가 하나만 있으므로 애너테이션을 생략할 수 있다.
  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

지금까지 우리가 진행했던 방법이 바로 생성자 주입이다.

- 생성자를 통해 의존 관게를 주입받는 방법
- 생성자 호출 시점에 딱 1번만 호출되는 것이 보장된다.
- 불변이고 필수인 의존 관계에 사용한다.
- 생성자가 딱 1개만 있으면 `@Autowired`를 생략해도 자동 주입된다.

제일 좋은 설계 습관은 제약이 있는 것이다. 그렇지 않으면 어디서 수정됐는지 알 수가 없다. 생성자는 어느 누구도 외부에서 수정할 수 없기 때문에 안전한 방법이다.

## 수정자(setter) 주입

```java

@Component
public class OrderServiceImpl implements OrderService {

  private MemberRepository memberRepository;
  private DiscountPolicy discountPolicy;

  // 애너테이션을 제거하면 주입되지 않는다.
  @Autowired
  public void setMemberRepository(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }

  @Autowired
  public void setDiscountPolicy(DiscountPolicy discountPolicy) {
    this.discountPolicy = discountPolicy;
  }
}

```

- 필드 값을 변경하는 수정자 메서드 setter를 통해 의존 관게를 주입하는 방법
- 선택적이고 변경 가능성이 있는 의존 관계에 사용한다.
    - 중간에 인스턴스를 바꾸고 싶다면 수정자 메서드를 외부에서 호출하면 된다.
- 자바 빈 프로퍼티 규약의 수정자 메서드 방식을 사용한다.

생성자 주입은 객체를 생성할 때 바로 빈을 new로 생성해서 주입한다. 하지만 수정자 주입에서 스프링 컨테이너의 라이프 사이클은 두 단계로 이루어져있다.
먼저 `OrderServiceImpl` 등의 빈을 컨테이너에 등록한 후, 의존 관계를 주입하는 것이다.

`@Autowired`가 붙은 곳에 주입할 대상이 없다면 기본적으로 오류가 발생한다. 주입할 대상이 없어도 동작하게 하려면 `@Autowired(required = false)`로
지정한다.

### 자바 빈 프로퍼티

```java
class Data {

  private int age;

  public void setAge(int age) {
    this.age = age;
  }

  public int getAge() {
    return age;
  }
}
```

자바 진영에서는 과거부터 필드 값을 직접 변경하지 않고 setter와 getter로 수정하고 조회하는 규칙을 사용해왔다.

## 필드 주입

```java

@Component
public class OrderServiceImpl implements OrderService {

  @Autowired
  private MemberRepository memberRepository;

  @Autowired
  private DiscountPolicy discountPolicy;
}
```

- 필드에 바로 주입하는 방법
- 코드가 간결하다.
- 외부에서 변경이 불가능해서 테스트하기 힘들다.
- DI 프레임워크 없이는 아무것도 할 수 없다.

되도록 사용하지 말자.

```java

@Configuration
public class AutoAppConfig {

  @Autowired
  private MemberRepository memberRepository;

  @Autowired
  private DiscountPolicy discountPolicy;

  @Bean
  OrderService orderService() {
    return new OrderServiceImpl(memberRepository, discountPolicy);
  }
}
```

애플리케이션 실제 코드와 관계 없는 테스트 코드나 스프링 설정을 목적으로 하는 `@Configuration` 같이 특별한 용도로만 사용하자.

```java
public class OrderServiceTest {

  // 순수한 자바 코드에서는 NPE이 발생한다.
  @Test
  void fieldInjectionTest() {
    OrderServiceImpl orderService = new OrderServiceImpl();
    orderService.createOrder(1L, "itemA", 1000);
  }
}

```

![](../../.gitbook/assets/kimyounghan-spring-core-principle/07/screenshot%202021-04-11%20오후%207.03.34.png)

스프링 컨테이너가 없는 순수한 자바 테스트 코드에서는 작동할 수 없는 방식이다. `@Autowired`는 스프링 컨테이너가 있어야만 동작한다.`@SpringBootTest`처럼
스프링 컨테이너를 테스트에 통합한 경우에만 가능하다.

## 일반 메서드 주입

```java

@Component
public class OrderServiceImpl implements OrderService {

  private MemberRepository memberRepository;
  private DiscountPolicy discountPolicy;

  @Autowired
  public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

- 일반 메서드를 통해 주입 받는 방식
- 한 번에 여러 필드를 주입 받을 수 있다.
- 일반적으로 잘 사용하지 않는다.

---

의존 관계 자동 주입은 스프링 컨테이너가 관리하는 스프링 빈이어야만 동작한다. 스프링 빈이 아닌 클래스에서 `@Autowired`를 적용하면 아무 동작도 하지 않는다. 예제에서도 `OrderServiceImpl`가 스프링 빈이기 때문에 주입이 되는 것이다.

## 옵션 처리

주입할 스프링 빈이 없어도 동작해야 할 때가 있다. 그런데 `@Autowired`만 사용하면 required 옵션이 true이기 때문에 자동 주입 대상이 없으면 오류가 발생한다.

자동 주입 대상을 옵션으로 처리하는 방법은 세 가지가 있다.

### @Autowired(required=false)

- 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출되지 않는다.

### org.springframework.lang.@Nullable

- 자동 주입할 대상이 없으면 null이 입력된다.

### Optional<>

- 자동 주입할 대상이 없으면 `Optional.empty`가 입력된다.

```java
public class AutowiredTest {

  @Test
  void autowiredOption() {
    // TestBean을 스프링 빈으로 등록한다.
    AnnotationConfigApplicationContext ac =
            new AnnotationConfigApplicationContext(TestBean.class);
  }

  static class TestBean {

    // 호출 안됨
    // true면 찾을 수 없다고 오류가 뜬다.
    @Autowired(required = false)
    public void setNoBean1(Member member) {
      System.out.println("setNoBean1 = " + member);
    }

    // null 호출
    @Autowired
    public void setNoBean2(@Nullable Member member) {
      System.out.println("setNoBean2 = " + member);
    }

    // Optional.empty 호출
    @Autowired(required = false)
    public void setNoBean3(Optional<Member> member) {
      System.out.println("setNoBean3 = " + member);
    }

  }
}
```

![](../../.gitbook/assets/kimyounghan-spring-core-principle/07/screenshot%202021-04-11%20오후%207.26.38.png)

사용되는 `Member`는 스프링 빈이 아니므로 의존 관계를 주입할 수 없다. 

결과를 보면, `setNoBean1()`은 `@Autowired(required = false)`이므로 호출 자체가 되지 않았다. `@Nullable`과 `Optional`은 스프링 전반에 걸쳐서 지원된다. 생성자 자동 주입에서 특정 필드에 사용할 수 있다.