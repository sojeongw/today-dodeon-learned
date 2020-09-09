# AOP

## AOP가 필요한 상황

만약 모든 메서드의 호출 시간을 측정하고 싶다면 어떻게 해야할까?

![](../../.gitbook/assets/kimyounghan-spring-introduction/07/스크린샷%202021-03-09%20오전%209.15.42.png)

{% tabs %} {% tab title="MemberService.java" %}

```java

@Transactional
public class MemberService {

  /**
   * 회원가입
   */
  public Long join(Member member) {
    // 시간 측정하는 기능 추가
    long start = System.currentTimeMillis();
    try {
      validateDuplicateMember(member);
      memberRepository.save(member);
      return member.getId();
    } finally {
      long finish = System.currentTimeMillis();
      long timeMs = finish - start;
      System.out.println("join " + timeMs + "ms");
    }
  }

  /**
   *전체 회원 조회
   */
  public List<Member> findMembers() {
    // 반복해서 추가
    long start = System.currentTimeMillis();
    try {
      return memberRepository.findAll();
    } finally {
      long finish = System.currentTimeMillis();
      long timeMs = finish - start;
      System.out.println("findMembers " + timeMs + "ms");
    }
  }
}
```

{% endtab %} {% endtabs %}

- 회원 가입, 조회에서 시간 측정 기능은 핵심 관심 사항이 아니라 공통의 관심 사항이다.
- 이런 로직이 핵심 비즈니스 로직과 섞이면 유지보수가 어려워진다.
- 그렇다고 이걸 메서드로 따로 빼서 공통 로직으로 만드는 것도 어렵다.
- 만약 시간 측정 로직이 변경되면 모든 로직을 찾아가면서 변경해야 한다.

## AOP 적용

AOP는 공통의 관심 사항(cross-cutting concern)과 핵심 관심 사항(core concern)을 분리하기 위해 고안되었다.

![](../../.gitbook/assets/kimyounghan-spring-introduction/07/스크린샷%202021-03-09%20오전%209.25.13.png)

위의 그림처럼 원하는 곳에 공통 관심 사항을 적용할 수 있는 것이다.

{% tabs %} {% tab title="TimeTraceAop.java" %}

```java
@Component  // 스프링 빈으로 등록해줘야 한다.
@Aspect // AOP를 사용하기 위한 애너테이션을 추가한다.
public class TimeTraceAop {

  // 어디에 적용할지 결정한다.
  // hello.hellospring 패키지 아래에 있는 모든 클래스에 적용한다.
  @Around("execution(* hello.hellospring..*(..))")
  public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();
    System.out.println("START: " + joinPoint.toString());
    
    try {
      // 다음 메서드로 진행한다.
      return joinPoint.proceed();
    } finally {
      long finish = System.currentTimeMillis();
      long timeMs = finish - start;
      System.out.println("END: " + joinPoint.toString() + " " + timeMs + "ms");
    }
  }
}
```

{% endtab %}
{% tab title="SpringConfig.java" %}

```java

@Configuration
public class SpringConfig {

  private final MemberRepository memberRepository;

  public SpringConfig(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }

  @Bean
  public MemberService memberService() {
    return new MemberService(memberRepository);
  }
  
  // @Component 대신 AOP를 사용한다는 점을 보여주기 위해 직접 빈을 등록해 사용하는 게 좋다.
  @Bean
  public TimeTraceAop timeTraceAop() {
    return new TImeTraceAop();
  }
}
```

{% endtab %}{% endtabs %}

- 회원 가입, 조회 등 핵심 관심 사항과 시간 측정이라는 공통 관심 사항을 분리한다.
- 시간 측정 로직을 별도의 공통 로직으로 빼서 핵심 관심 사항을 깔끔하게 유지할 수 있다.
- 변경이 필요하면 이 로직만 변경할 수 있다.
- 원하는 적용 대상을 선택할 수 있다.

## 스프링의 AOP 동작 방식

### AOP 적용 전 의존 관계

![](../../.gitbook/assets/kimyounghan-spring-introduction/07/스크린샷%202021-03-09%20오전%209.37.06.png)

컨트롤러가 서비스를 의존하면서 필요한 메서드를 호출한다.

### AOP 적용 후 의존 관계

![](../../.gitbook/assets/kimyounghan-spring-introduction/07/스크린샷%202021-03-09%20오전%209.37.12.png)

프록시라는 가짜 서비스를 만들어서, 스프링이 실행될 때 컨테이너에 진짜 스프링 빈 말고 가짜 스프링 빈을 앞에 세운다. AOP 로직을 다 실행하고 `joinPoint.proceed()`로 가짜 스프링 빈이 끝나면 진짜 스프링 빈을 호출한다. 따라서 컨트롤러는 진짜 서비스가 아니라 가짜 서비스를 호출한다.

```java
@Controller
public class MemberController {

	private MemberService memberService;
	
	@Autowired
    public MemberController(MemberService memberService) {
	  this.memberService = memberService;

	  // 프록시 동작을 확인해본다.
      System.out.println("memberService = " + memberService.getClass());
    }
}
```

![](../../.gitbook/assets/kimyounghan-spring-introduction/07/스크린샷%202021-03-09%20오전%209.47.12.png)

출력해보면 `EnhancerBySpringCGLIB`라는 코드 복제 기술이 출력된다. 실제 프록시가 주입되는지 확인한 것이다.

### AOP 적용 전 전체 그림

![](../../.gitbook/assets/kimyounghan-spring-introduction/07/스크린샷%202021-03-09%20오전%209.38.20.png)

AOP를 적용하기 전에는 이런 그림이었다.

### AOP 적용 후 전체 그림

![](../../.gitbook/assets/kimyounghan-spring-introduction/07/스크린샷%202021-03-09%20오전%209.38.26.png)

적용 후에는 프록시 호출하는 흐름으로 동작한다. 스프링 컨테이너에서 빈을 관리하면 가짜를 만들어서 주입하는 게 쉬워진다. DI의 장점이다. 컨트롤러는 '뭔진 모르겠고 그냥 받아서 쓸게'가 가능해진다. 그래서 AOP가 가능한 것이다.