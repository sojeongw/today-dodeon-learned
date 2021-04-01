# 스프링 빈 설정 메타 정보

스프링이 다양한 설정 형식을 지원할 수 있는 것은 `BeanDefinition`이라는 추상화 덕분이다. 역할과 구현을 개념적으로 나눴다고 보면 된다.

- XML을 읽어서 BeanDefinition을 만든다.
- 자바 코드를 읽어서 BeanDefinition을 만든다.

그럼 스프링 컨테이너는 자바 코드인지 XML인지 알 필요 없이 오직 `BeanDefinition`만 알면 된다. 이걸 기반으로 빈 정보를 생성한다.

`BeanDefinition`을 빈 설정 메타 정보라고 하는데 `@Bean`, `<bean>` 당 각각 하나씩 메타 정보가 생성된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%205.49.43.png)

스프링 컨테이너 자체는 `BeanDefinition`만 의존한다. 무엇으로 설정된 정보인지는 신경쓰지 않는다. 추상화에만 의존하도록 설계한 것이다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%205.50.17.png)

`AnnotationConfigApplicationContext`는 `AnnotatedBeanDefinitionReader`를 사용해 `AppConfig.class`를 읽고 `BeanDefinition`을 생성한다.

`GenericXmlApplicationContext`는 `XmlBeanDefinitionReader`를 사용해 `appConfig.xml` 설정 정보를 읽고 `BeanDefinition`을 생성한다.

새로운 형식의 설정 정보가 추가되면 `xxxBeanDefinitionReader`를 사용해 `BeanDefinition`을 생성한다.

## BeanDefinition 정보

{% tabs %} {% tab title="BeanDefinitionTest.java" %}

```java
public class BeanDefinitionTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

  @Test
  @DisplayName("빈 설정 메타 정보 확인")
  void findApplicationBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
      BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

      if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION){
        System.out.println("beanDefinitionName = " + beanDefinitionName + ", beanDefinition = " + beanDefinition);
      }
    }

  }

}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%207.47.38.png)

### BeanClassName

- 생성할 빈의 클래스 명
- 자바 설정 처럼 팩토리 역할의 빈을 사용하면 없다.

### factoryBeanName

- 팩토리 역할의 빈을 사용할 경우 지정되는 이름
- ex) appConfig

### factoryMethodName

- 빈을 생성할 팩토리 메서드
- ex) memberService

`AppConfig`처럼 메서드를 통해 빈을 등록하는 방식을 팩토리 메서드 방식이라고 한다. 외부에서 해당 메서드를 호출해서 주입하는 방식이다. 

### Scope

- 싱글톤(기본값)

### lazyInit

- 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생성을 지연 처리 하는지의 여부

### InitMethodName

- 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명 

### DestroyMethodName

- 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명

### Constructor arguments, Properties

- 의존 관계 주입에서 사용
- 자바 설정처럼 팩토리 역할의 빈을 사용한다면 없다.

## 정리

`new BeanDefinition(...)`처럼 직접 생성해서 스프링 컨테이너에 등록할 수도 있다. 하지만 실무에서는 `BeanDefinition`을 직접 정의하거나 사용할 일은 거의 없으므로 어려우면 그냥 넘어가면 된다.

`BeanDefinition`을 너무 깊이 있게 이해하려고 하기 보다는, 스프링이 다양한 형태의 설정 정보를 `BeanDefinition`으로 추상화해서 사용한다는 것 정도만 알면 된다.

가끔 스프링 코드나 관련 오픈 소스에서 `BeanDefinition`이 보일 때가 있는데 이때 이러한 매커니즘을 떠올리면 된다.