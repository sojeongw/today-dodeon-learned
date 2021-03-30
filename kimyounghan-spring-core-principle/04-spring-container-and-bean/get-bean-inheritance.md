# 상속 관계일 때 스프링 빈 조회

부모 타입으로 조회하면 자식 타입도 함께 조회한다. 그래서 모든 자바 객체의 최고 부모인 `Object` 타입으로 조회하면 모든 스프링 빈을 조회한다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%205.01.41.png)

상속 관계가 위와 같다면 1번 타입으로 조회했을 떄 모든 빈이 조회된다. 그 다음 번호들도 자기 자식만큼 나온다.

{% tabs %} {% tab title="ApplicationContextInheritanceTest.java" %}

```java
public class ApplicationContextInheritanceTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

  @Test
  @DisplayName("부모 타입으로 조회 시 자식이 둘 이상 있으면 중복 오류가 발생한다.")
  void findBeanByParentTypeDuplicate() {
    assertThrows(NoUniqueBeanDefinitionException.class, () -> ac.getBean(DiscountPolicy.class));
  }

  @Configuration
  static class TestConfig {

    @Bean
    public DiscountPolicy rateDiscountPolicy() {
      return new RateDiscountPolicy();
    }

    @Bean
    public DiscountPolicy fixDiscountPolicy() {
      return new FixDiscountPolicy();
    }
  }

}
```

{% endtab %} {% endtabs %}

자식이 둘 이상이라면 중복 오류가 발생한다.

참고로 `TestConfig`의 메서드들은 구체 타입이 아니라 인터페이스를 반환하도록 정의되어 있는데, 물론 구체 타입으로 반환해도 되지만 역할과 구현을 쪼개야 하기 때문에 이렇게 선언한 것이다.

## 빈 이름으로 지정해서 조회

{% tabs %} {% tab title="ApplicationContextInheritanceTest.java" %}

```java
public class ApplicationContextInheritanceTest {

  @Test
  @DisplayName("부모 타입으로 조회 시 자식이 둘 이상 있으면 빈 이름을 지정해 조회한다.")
  void findBeanByParentTypeBeanName() {
    DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
    assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
  }

}
```

{% endtab %} {% endtabs %}

## 특정 하위 타입으로 조회

{% tabs %} {% tab title="ApplicationContextInheritanceTest.java" %}

```java
public class ApplicationContextInheritanceTest {
  
  @Test
  @DisplayName("특정 하위 타입으로 조회한다.")
  void findBeanBySubType() {
    DiscountPolicy rateDiscountPolicy = ac.getBean(RateDiscountPolicy.class);
    assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
  }

}
```

{% endtab %} {% endtabs %}

이 방법은 구체 클래스로 조회하기 때문에 좋은 방법은 아니다.

## 부모 타입으로 모든 빈 조회

{% tabs %} {% tab title="ApplicationContextInheritanceTest.java" %}

```java
public class ApplicationContextInheritanceTest {

  @Test
  @DisplayName("부모 타입으로 모든 빈을 조회한다.")
  void findBeanByParentType() {
    Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);

    for (String key : beansOfType.keySet()) {
      System.out.println("key = " + key + ", value = " + beansOfType.get(key));
    }

    assertThat(beansOfType.size()).isEqualTo(2);
  }

}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%205.15.55.png)

참고로 출력 로직은 공부를 위한 것이지 실제 테스트 코드에 넣으면 안된다. 테스트 통과 여부를 시스템이 결정하도록 해야하기 때문이다.

## Object 타입으로 모든 빈 조회

{% tabs %} {% tab title="ApplicationContextInheritanceTest.java" %}

```java
public class ApplicationContextInheritanceTest {

  @Test
  @DisplayName("Object 타입으로 모든 빈을 조회한다.")
  void findBeanByObject() {
    Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);

    for (String key : beansOfType.keySet()) {
      System.out.println("key = " + key + ", value = " + beansOfType.get(key));
    }
  }

}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%205.17.36.png)

스프링이 내부적으로 사용하는 빈까지 모두 출력되었다.

---

실제 프로젝트 코드에는 위와 같이 `ApplicationContext`를 직접 불러오는 일은 거의 없다. 하지만 부모가 여러 자식을 가질 때 어떻게 하는지 등을 알고 있어야 다음 개념을 이해할 수 있으므로 설명했다.