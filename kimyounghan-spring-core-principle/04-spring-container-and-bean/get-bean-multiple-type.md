# 동일 타입이 둘 이상일 때 스프링 빈 조회

같은 타입의 스프링 빈이 둘 이상이면 타입으로 조회할 때 오류가 발생한다. 이 때는 빈 이름을 지정해야 한다. 해당 타입의 모든 빈을 조회하려면 `ac.getBeansOfType`을 사용한다.

{% tabs %} {% tab title="ApplicationContextSameBeanTest.java" %}

```java
public class ApplicationContextSameBeanTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

  @Test
  @DisplayName("타입으로 조회할 때 둘 이상이 있으면 중복 오류가 발생한다.")
  void findBeanByTypeDuplicate() {
    MemberRepository bean = ac.getBean(MemberRepository.class);

  }

  // 테스트용 설정 클래스
  @Configuration
  // static: 클래스 안에 클래스를 만들어 여기서만 쓸 때 사용한다.
  static class SameBeanConfig{
    @Bean
    public MemberRepository memberRepository1() {
      // MemoryMemberRepository("10")처럼 다른 파라미터를 넘겨 빈이 생성될 수 있으므로
      // 이렇게 여러 개로 정의하는 건 틀린 게 아니다.
      return new MemoryMemberRepository();
    }

    @Bean
    public MemberRepository memberRepository2() {
      return new MemoryMemberRepository();
    }
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%204.46.37.png)

같읕 타입을 반환하는 빈이 두 개면 `NoUniqueBeanDefinitionException` 예외가 발생한다.

## 빈 이름을 지정해서 조회

{% tabs %} {% tab title="ApplicationContextSameBeanTest.java" %}

```java
public class ApplicationContextSameBeanTest {

  @Test
  @DisplayName("타입으로 조회할 때 같은 타입이 둘 이상이면 빈 이름을 지정해서 불러온다.")
  void findBeanByName() {
    MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
    assertThat(memberRepository).isInstanceOf(MemberRepository.class);
  }
}
```

{% endtab %} {% endtabs %}

특정 빈 이름을 써서 불러오면 된다.

## 특정 타입 모두 조회

{% tabs %} {% tab title="ApplicationContextSameBeanTest.java" %}

```java
public class ApplicationContextSameBeanTest {

  @Test
  @DisplayName("특정 타입을 모두 조회한다.")
  void findAllBeanByType() {
    Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
    
    for (String key : beansOfType.keySet()) {
      System.out.println("key = " + key + " value = " + beansOfType.get(key));
    }

    System.out.println("beansOfType = " + beansOfType);
    assertThat(beansOfType.size()).isEqualTo(2);
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%204.57.23.png)

두 가지 빈이 있는 것을 알 수 있다.