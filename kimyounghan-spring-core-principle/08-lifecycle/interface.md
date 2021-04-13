# 인터페이스 InitializingBean, DisposableBean

{% tabs %} {% tab title="NetworkClient.java" %}

```java
public class NetworkClient implements InitializingBean, DisposableBean {

  private String url;

  public NetworkClient() {
    System.out.println("생성자 호출, url = " + url);
  }

  ...

  // InitializingBean
  // 프로퍼티가 세팅이 끝나면 = 의존 관게 주입이 끝나면 호출한다.
  @Override
  public void afterPropertiesSet() throws Exception {
    connect();
    call("초기화 연결 메시지");
  }

  // DisposableBean
  // 빈이 종료될 때 호출된다.
  @Override
  public void destroy() throws Exception {
    disconnect();
  }
}

```

{% endtab %} {% tab title="BeanLifeCycleTest.java" %}

```java
public class BeanLifeCycleTest{

  @Test
  public void lifeCycleTest() {
    ConfigurableApplicationContext ac =  new AnnotationConfigApplicationContext(LifeCycleConfig.class);
    NetworkClient client = ac.getBean(NetworkClient.class);
    ac.close();
  }

  @Configuration
  static class LifeCycleConfig {
    @Bean
    public NetworkClient networkClient() {
      NetworkClient networkClient = new NetworkClient();
      networkClient.setUrl("http://hello-spring.dev");
      return networkClient;
    }
  }

}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/08/screenshot%202021-04-13%20오전%209.13.02.png)

실행하면 빈이 생성된 뒤 url이 정상적으로 초기화된 것을 확인할 수 있다. 이후 스프링 컨테이너가 종료되자 소멸 메서드가 호출되었다.

## 단점

- 스프링 전용 인터페이스이므로 코드가 스프링 전용 인터페이스에 의존한다.
- 초기화, 소멸 메서드의 이름을 변경할 수 없다.
- 개발자가 직접 코드를 고칠 수 없는 외부 라이브러리에는 적용할 수 없다.

인터페이스를 사용하는 초기화, 종료 방법은 초창기 방식이기 때문에 지금은 거의 사용하지 않는다.