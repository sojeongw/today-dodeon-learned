# 컴포넌트 스캔

지금까지는 스프링 빈을 등록할 때 자바 코드의 `@Bean`이나 xml의 `<bean>`을 통해 직접 등록할 스프링 빈을 나열했다. 하지만 이런 작업은 설정 파일이 커질 뿐더러 귀찮고 실수가 발생할 수 있다.

그래서 스프링은 설정 정보 없이도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔 기능을 제공한다. 의존 관계를 자동 주입하는 `@Autowired`도 제공한다.

{% tabs %} {% tab title="AutoAppConfig.java" %}

```java
@Configuration
// @Component가 붙은 객체를 찾아서 스프링 빈으로 등록해준다.
@ComponentScan(
    // AppConfig.java와의 충돌을 피하기 위해
    // Configuration 타입의 애너테이션이 달린 것은 제외한다.
    // @Configuration에 들어가보면 @Component가 달려있어 스캔 대상이 되기 때문이다.
    excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
)
public class AutoAppConfig {
}
```

{% endtab %} {% endtabs %}

컴포넌트 스캔을 사용하려면 `ComponentScan`을 설정 정보에 붙여준다. 기존 `AppConfig`와는 다르게 `@Bean`으로 등록한 클래스가 없는 걸 볼 수 있다.

컴포넌트 스캔을 사용하면 `@Configuration`도 `@Component`가 붙어있어 자동으로 등록된다. 따라서 다른 설정 정보와 충돌을 피하기 위해 `excludeFilters`로 스캔 대상에서 제외한다. 

```java
@Component
public class MemoryMemberRepository implements MemberRepository {}

@Component
public class RateDiscountPolicy implements DiscountPolicy{}

@Component
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;

  // ac.getBean(MemberRepository.class)와 같은 동작을 한다.
  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository,
      DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}

@Component
public class MemberServiceImpl implements MemberService {

  private final MemberRepository memberRepository;

  @Autowired
  public MemberServiceImpl(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }
}
```

이제 사용하는 구현체에 `@Component`를 붙여준다. 이제는 `AppConfig`처럼 의존 관계를 반환해서 등록하는 메서드가 없기 때문에, 생성자에 `@Autowired`를 붙여 여러 의존 관계를 한 번에 주입받는다.

```java
public class AutoAppConfigTest {

  @Test
  void basicScan() {
    AnnotationConfigApplicationContext ac =
        new AnnotationConfigApplicationContext(AutoAppConfig.class);

    MemberService memberService = ac.getBean(MemberService.class);

    assertThat(memberService).isInstanceOf(MemberService.class);
  }

}
```

`AnnotationConfigApplicationContext`를 사용하는 것은 기존과 동일하다. 설정 정보로 `AutoAppConfig`를 넘겨주면 기존과 같이 잘 동작한다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%205.12.35.png)

로그를 보면 컴포넌트 스캔으로 의존성 주입이 잘 되는 것을 볼 수 있다.

## @ComponentScan의 동작 방식

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%205.14.49.png)

`@ComponentScan`이 되어있으면 스프링 컨테이너가 `@Component`가 붙은 모든 클래스를 스프링 빈으로 등록한다.

### Bean 이름 전략

- 기본값
    - 앞 글자만 소문자로 바꾼 클래스 이름
    - `MemberServiceImpl` 클래스라면 `memberServiceImpl`
- 직접 지정
    - @Component("memberService")

## @Autowired의 동작 방식

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%205.15.02.png)

생성자에 `@Autowired`를 지정하면 스프링 컨테이너가 자동으로 해동 스프링 빈을 찾아서 주입한다.

### 조회 전략

- **타입**이 같은 빈을 찾아서 주입한다.
- `getBean(MemberRepository.class)`와 동일하다고 이해하면 된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/05/screenshot%202021-04-11%20오후%205.15.08.png)

생성자에 파라미터가 두 개 이상이어도 다 찾아서 자동으로 주입한다.