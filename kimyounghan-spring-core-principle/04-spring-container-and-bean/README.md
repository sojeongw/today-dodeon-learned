# 스프링 컨테이너와 스프링 빈

```java
public class OrderApp {

  public static void main(String[] args) {
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
  }
}
```

`ApplicationContext`는 스프링 컨테이너라고 하는 인터페이스다. 이 인터페이스를 구현한 클래스 중 하나가 위의 `AnnotationConfigApplicationContext`다.

스프링 컨테이너는 XML 혹은 애터에시녀 기반 자바 설정 클래스로 만들 수 있다. 직전에 `AppConfig`를 사용했던 방식이 애너테이션 기반의 자바 설정 클래스로 스프링 컨테이너를 만든 것이다.

스프링 컨테이너는 `BeanFactory`와 `ApplicationContext`로 구분해서 얘기한다. 추후 설명하겠지만 `BeanFactory`를 직접 사용하는 경우는 거의 없으므로 일반적으로 `ApplicationContext`를 스프링 컨테이너라고 한다. 

## 스프링 컨테이너의 생성 과정
### 1. 스프링 컨테이너 생성

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%201.11.34.png)

```
new AnnotationConfigApplicationContext(AppConfig.class);
```

스프링 컨테이너를 생성할 떄 구성 정보를 지정해줘야 한다. 여기서는 `AppConfig.class`를 구성 정보로 지정했다.

스프링 컨테이너에는 키가 bean 이름, 값이 bean 객체로 되어있는 스프링 저장소가 들어있다.

### 2. 스프링 빈 등록

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%201.12.28.png)

넘긴 설정 정보를 하나씩 살펴보면서 `@Bean`이 붙은 함수를 모두 호출한 뒤 메서드 이름을 빈 이름으로, 반환하는 객체를 빈 객체로 등록한다. 이렇게 만들어진 것을 스프링 빈이라고 한다.

```
@Bean(name = "memberService2")
```

빈 이름의 기본 값은 메서드 이름이지만 직접 부여할 수도 있다.

참고로 빈 이름은 항상 다르게 부여해야 한다. 같은 이름을 부여하면 다른 빈이 무시되거나, 기존 빈을 덮어버리거나, 설정에 따라 오류가 발생한다.

### 3. 스프링 빈 의존 관계 설정

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%201.12.39.png)

스프링 컨테이너가 스프링 빈을 등록한 뒤에는 의존 관계를 넣어주어야 한다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%201.12.44.png)

설정 정보를 참고해 `memberService`와 `orderService`의 의존 관계에 `memberRepository`를 넣어준다. 단순히 자바 코드를 호출하는 것 같지만 차이가 있다. 이 차이는 뒤에 싱글톤 컨테이너에서 설명한다.

스프링은 빈 생성과 의존 관계 주입 단계가 나눠져 있다. 여기서는 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존 관계 주입도 한번에 처리 되는 것 처럼 설명했다. 자세한 내용은 의존 관계 자동 주입에서 다시 설명한다.
