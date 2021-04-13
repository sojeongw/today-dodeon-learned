# 빈 등록 초기화, 소멸 메서드 지정

설정 정보에 `@Bean(initMethod = "init", destroyMethod = "close")`를 쓰면 초기화, 소멸 메서드를 지정할 수 있다.

{% tabs %} {% tab title="NetworkClient.java" %}

```java
public class NetworkClient {

  private String url;

  public NetworkClient() {
    System.out.println("생성자 호출, url = " + url);
  }

  ...

  // 메서드 이름을 임의로 변경한다.
  public void init() throws Exception {
    connect();
    call("초기화 연결 메시지");
  }

  public void close() throws Exception {
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
    // 각 단계에서 호출할 메서드의 이름을 넣어준다.
    @Bean(initMethod = "init", destroyMethod = "close")
    public NetworkClient networkClient() {
      NetworkClient networkClient = new NetworkClient();
      networkClient.setUrl("http://hello-spring.dev");
      return networkClient;
    }
  }

}

```

{% endtab %} {% endtabs %}

설정 정보에서 메서드만 지정해주면 이전과 동일한 결과가 나온다.

## 특징

- 메서드 이름을 자유롭게 줄 수 있다.
- 스프링 빈이 스프링 코드에 의존하지 않는다.
- 코드가 아니라 설정 정보를 이용하므로 코드를 고칠 수 없는 외부 라이브러리에도 적용할 수 있다.

## destroyMethod

라이브러리는 대부분 `close`, `shutdown`이라는 이름의 종료 메서드를 사용한다. `@Bean`의 `destroyMethod`는 기본값이 `inferred`로 되어있다. 이 기능은 `close`, `shutdown`이라는 이름의 메서드를 추론해서 자동으로 호출한다.  따라서 직접 스프링 빈으로 등록하고 싶지 않으면 종료 메서드를 따로 적어주지 않아도 종료 메서드를 잘 호출해서 동작한다. 만약 추론 기능을 사용하기 싫다면 `destroyMethod=""`처럼 빈 공백을 지정하면 된다.