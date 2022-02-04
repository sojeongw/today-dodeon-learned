# 필터와 중복 등록

## 필터

- includeFilters
    - 컴포넌트 스캔 대상을 추가로 지정한다.
- excludeFilters
    - 컴포넌트 스캔에서 제외할 대상을 지정한다.

{% tabs %} {% tab title="MyIncludeComponent.java" %}

```java
// TYPE: 클래스 레벨에 붙게 설정
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyIncludeComponent {

}
```

{% endtab %} {% tab title="MyExcludeComponent.java" %}

```java

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyExcludeComponent {

}
```

{% endtab %} {% tab title="BeanA.java" %}

```java

@MyIncludeComponent
public class BeanA {

}
```

{% endtab %} {% tab title="BeanB.java" %}

```java

@MyExcludeComponent
public class BeanB {

}
```

{% endtab %} {% tab title="ComponentFilterAppConfigTest.java" %}

```java
public class ComponentFilterAppConfigTest {

  @Test
  void filterScan() {
    AnnotationConfigApplicationContext ac =
        new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);

    // 성공
    BeanA beanA = ac.getBean("beanA", BeanA.class);
    assertThat(beanA).isNotNull();

    // NoSuchBeanDefinitionException: No bean named 'beanB' available
    assertThrows(NoSuchBeanDefinitionException.class, () -> ac.getBean("beanB", BeanB.class));
  }

  @Configuration
  @ComponentScan(
      includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
      excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
  )
  static class ComponentFilterAppConfig {

  }
}
```

{% endtab %} {% endtabs %}

임의의 애너테이션을 정의해서 filter에 추가하면 exclude된 애너테이션이 달린 `beanB`는 스프링 빈에 등록되지 않는다.

## FilterType 옵션

### ANNOTATION

- 애너테이션을 인식해서 동작한다.
- ex) org.example.SomeAnnotation

기본 값이기 때문에 생략하고 그냥 `includeFilters = classes = MyIncludeComponent.class)`만 해줘도 동작한다.

### ASSIGNABLE_TYPE

- 지정한 타입과 자식 타입을 인식해서 동작한다.
- ex) org.example.SomeClass

```java
@ComponentScan(
    includeFilters = {
        @Filter(type = FilterType.ANNOTATION, classes =
            MyIncludeComponent.class),
    },
    excludeFilters = {
        @Filter(type = FilterType.ANNOTATION, classes =
            MyExcludeComponent.class),
        @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = BeanA.class)
    }
)
```

만약 `BeanA`도 빼고 싶다면 위와 같이 추가하면 된다.

### ASPECTJ

- AspectJ 패턴을 사용한다.
- ex) org.example..*Service+

### REGEX

- 정규 표현식을 사용한다.
- ex) org\.example\.Default.*

### CUSTOM

- `TypeFilter`라는 인터페이스를 구현해서 처리한다.
- ex) org.example.MyTypeFilter

--- 

대부분 `@Component`면 충분하기 때문에, 필터를 사용할 일이 거의 없다. 특히 최근 스프링 부트는 컴포넌트 스캔을 기본으로 제공하기 때문에 기본 설정에 최대한 맞춰
사용하는 것을 권장한다.

## 중복 등록과 충돌

컴포넌트 스캔에서 같은 빈 이름을 등록하면 어떻게 될까?

### 자동 빈 등록 + 자동 빈 등록

컴포넌트 스캔으로 스프링 빈이 자동 등록될 때 이름이 같은 게 있을 경우 스프링이 `ConflictingBeanDefinitionException`를 발생시킨다.

### 수동 빈 등록 + 자동 빈 등록

{% tabs %} {% tab title="MemoryMemberRepository.java" %}

```java

@Component
public class MemoryMemberRepository implements MemberRepository {

}
```

{% endtab %} {% tab title="AutoAppConfig.java" %}

```java

@Configuration
@ComponentScan
public class AutoAppConfig {

  @Bean(name = "memoryMemberRepository")
  public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/06/screenshot%202021-04-11%20오후%206.02.33.png)

이 경우에는 수동 빈이 자동 빈을 오버라이딩 해버리기 때문에 수동 빈 등록이 우선권을 가진다.

개발자가 의도적으로 했을 경우 자동보다는 수동이 우선권을 가지는 것이 좋다. 하지만 보통은 여러 설정이 꼬이면서 이런 결과가 나온다. 이런 애매한 버그는 항상 잡기 어렵다. 

![](../../.gitbook/assets/kimyounghan-spring-core-principle/06/screenshot%202021-04-11%20오후%206.06.33.png)

그래서 최근 스프링 부트에서는 수동 빈 등록과 자동 빈 등록이 충돌나면 위의 오류가 발생하도록 기본 값을 바꾸었다. 만약 바꾸고 싶다면 `application.properties`에 `spring.main.allow-bean-definition-overriding=true`를 설정하면 된다.