# 다양한 설정 형식

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%205.33.04.png)

스프링 컨테이너는 다양한 형식의 설정 정보를 받을 수 있도록 유연하게 설계되어 있다.

## 애너테이션 기반 자바 코드 설정

지금까지 했던 방식이 이것이다. `new AnnotationConfigApplicationContext(AppConfig.class)`로 자바 코드로 된 설정 코드를 넘긴다.

## XML 설정

스프링 부트가 보편화되면서 xml 설정은 잘 사용하지 않는다. 컴파일 없이 빈 설정 정보를 바꿀 수 있는 장점이 있다. `GenericXmlApplictionContext`에 xml 설정 파일을 넘기는 방식이다.

{% tabs %} {% tab title="XmlAppContext.java" %}

```java
public class XmlAppContext {

  @Test
  void xmlAppContext() {
    ApplicationContext ac =
        new GenericXmlApplicationContext("appConfig.xml");

    MemberService memberService =
        ac.getBean("memberService", MemberService.class);

    assertThat(memberService).isInstanceOf(MemberService.class);
  }
}

```

{% endtab %} {% tab title="appConfig.xml" %}

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <bean id="memberService" class="hello.core.member.MemberServiceImpl">
    <constructor-arg name="memberRepository" ref="memberRepository"/>
  </bean>
  
  <bean id="memberRepository"
    class="hello.core.member.MemoryMemberRepository"/>
  
  <bean id="orderService" class="hello.core.order.OrderServiceImpl">
    <constructor-arg name="memberRepository" ref="memberRepository"/>
    <constructor-arg name="discountPolicy" ref="discountPolicy"/>
  </bean>
  
  <bean id="discountPolicy" class="hello.core.discount.RateDiscountPolicy"/>
</beans>
```

{% endtab %} {% endtabs %}

`appConfig.xml`과 `AppConfig.java`를 비교하면 비슷할 것이다.