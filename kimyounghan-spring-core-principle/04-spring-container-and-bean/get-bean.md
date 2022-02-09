# 스프링 빈 조회

스프링 컨테이너에 실제 스프링 빈이 잘 등록되었는지 확인해보자.

{% tabs %} {% tab title="ApplicationContextInfoTest.java" %}

```java
public class ApplicationContextInfoTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

  @Test
  @DisplayName("모든 빈 출력하기")
  void findAllBean() {
    // 스프링에 등록된 모든 빈 이름을 조회한다.
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinition : beanDefinitionNames) {
      // 빈 이름으로 빈 객체(인스턴스)를 조회한다.
      Object bean = ac.getBean(beanDefinition);
      System.out.println("beanDefinition = " + beanDefinition + ", object = " + bean);
    }
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%203.20.18.png)

스프링 컨테이너에 있는 모든 빈 정보가 출력되었다.

{% tabs %} {% tab title="ApplicationContextInfoTest.java" %}

```java
public class ApplicationContextInfoTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

  @Test
  @DisplayName("애플리케이션 빈 출력하기")
  void findApplicationBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
      BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

      // ROLE_APPLICATION: 직접 등록한 애플리케이션 빈
      // ROLE_INFRASTRUCTURE: 스프링 내부에서 사용하는 빈
      if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
        Object bean = ac.getBean(beanDefinitionName);
        System.out.println("name = " + beanDefinitionName + ", object = " + bean);
      }
    }
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%203.19.32.png)

스프링 내부에서 쓰는 빈은 제외하고 내가 직접 등록한 빈만 출력할 수도 있다.

```
ac.getBean(빈 이름, 타입);
ac.getBean(타입);
```

스프링 컨테이너에서 스프링 빈을 찾는 가장 기본적인 조회 방법은 위와 같다. 조회 대상 스프링 빈이
없으면 `NoSuchBeanDefinitoinException: No bean named '***' available` 예외가 발생한다.

## 빈 이름으로 조회

{% tabs %} {% tab title="ApplicationContextBasicFindTest.java" %}

```java
public class ApplicationContextBasicFindTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

  @Test
  @DisplayName("빈 이름으로 조회")
  void findBeanByName() {
    MemberService memberService = ac.getBean("memberService", MemberService.class);
    System.out.println("memberService = " + memberService);
    System.out.println("memberService.getClass() = " + memberService.getClass());

    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%204.27.44.png)

`memberService`빈을 불러와 `MemberServiceImpl`의 인스턴스라는 것을 확인했다.

## 이름 없이 타입으로 조회

{% tabs %} {% tab title="ApplicationContextBasicFindTest.java" %}

```java
public class ApplicationContextBasicFindTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

  @Test
  @DisplayName("이름 없이 타입으로만 조회")
  void findBeanByType() {
    MemberService memberService = ac.getBean(MemberService.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
  }
}
```

{% endtab %} {% endtabs %}

## 구체 타입으로 조회

{% tabs %} {% tab title="ApplicationContextBasicFindTest.java" %}

```java
public class ApplicationContextBasicFindTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

  @Test
  @DisplayName("구체 타입으로 조회")
  void findBeanByName2() {
    // 기본적으로 인터페이스로 조회하면 인터페이스의 구현체가 조회 대상이 된다.
    // 그래서 이렇게 직접 구현체로 빈을 가져올 수도 있다.
    MemberService memberService = ac.getBean(MemberServiceImpl.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
  }
}
```

{% endtab %} {% tab title="AppConfig.java" %}

```java
@Configuration
public class AppConfig {

  @Bean
  public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
  }
}
```

{% endtab %} {% endtabs %}

`memberService`가 `AppConfig`에서 `MemberService`를 반환한다고 메서드가 정의되어 있어도, 실제로는 `MemberServiceImpl`라는 인스턴스 타입을 보고 결정하기 때문에 구체 클래스로도 조회할 수 있다.

하지만 이 방법은 좋지 않다. 역할과 구현을 구분하고 역할에 의존해야 하기 때문이다.

## 존재하지 않는 빈 조회

{% tabs %} {% tab title="ApplicationContextBasicFindTest.java" %}

```java
public class ApplicationContextBasicFindTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

  @Test
  @DisplayName("빈 이름으로 조회 실패")
  void findBeanByNameFail() {
    assertThrows(NoSuchBeanDefinitionException.class,
        () -> ac.getBean("없는 빈", MemberServiceImpl.class));
  }
}
```

{% endtab %} {% endtabs %}
