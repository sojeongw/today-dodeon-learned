# 애너테이션 @PostConstruct, @PreConstruct

{% tabs %} {% tab title="NetworkClient.java" %}

```java
public class NetworkClient {

  private String url;

  public NetworkClient() {
    System.out.println("생성자 호출, url = " + url);
  }

  ...

  @PostConstruct
  public void init() throws Exception {
    connect();
    call("초기화 연결 메시지");
  }

  @PreDestroy
  public void close() throws Exception {
    disconnect();
  }
}
```

{% endtab %} {% tab title="BeanLifeCycleTest.java" %}

```java
public class BeanLifeCycleTest {

  @Test
  public void lifeCycleTest() {
    ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(
        LifeCycleConfig.class);
    NetworkClient client = ac.getBean(NetworkClient.class);
    ac.close();
  }

  @Configuration
  static class LifeCycleConfig {

    // 빈 등록은 해줘야 돌아간다.
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

설정 파일에서 해당 빈을 등록해주면 애너테이션에 붙은 메서드를 생명 주기에 따라 실행한다.

## 특징

- 최신 스프링에서 가장 권장하는 방법
- 애너테이션 하나만 붙이면 되므로 편리하다.
- `javax.annotaion.PostConstruct` 패키지 소속이다.
    - 스프링에 종속적인 게 아니라 JSR-250이라는 자바 표준이다.
    - 따라서 스프링이 아닌 다른 컨테이너에서도 동작한다.
- 빈을 등록하는 게 아니므로 컴포넌트 스캔 기능과 잘 어울린다.

## 단점

- 코드를 고치는 게 아니므로 외부 라이브러리에 적용할 수 없다.
    - 이때는 직전에 배운 @Bean의 옵션을 사용하자.