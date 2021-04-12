# 빈 생명 주기 콜백

보통 TCP/IP 통신을 하면 handshake 등 시간이 오래 걸리기 때문에 서버를 띄울 때 DB 커넥션도 같이 받아놓고 빨리 응답을 주도록 한다. 이런 커넥션은 애플리케이션이
종료될 때 함께 안전하게 종료해줘야 한다. 이렇게 하려면 객체의 초기화와 종료 작업이 필요하다.

외부 네트워크에 미리 연결하는 객체를 하나 생성한다고 가정해보자. `NetworkClient`는 애플리케이션 시작 시점에 `connnect()`를 호출해서 미리 연결을
맺고 `disconnect()`로 연결을 끊는다.

{% tabs %} {% tab title="NetworkClient.java" %}

```java
public class NetworkClient {

  private String url;

  public NetworkClient() {
    System.out.println("생성자 호출, url = " + url);
    connect();
    call("초기화 연결 메시지");
  }

  public void setUrl(String url) {
    this.url = url;
  }

  // 서비스를 시작할 때 호출하는 메서드
  public void connect() {
    System.out.println("connected url = " + url);
  }

  // 메시지를 보내는 메서드
  public void call(String message) {
    System.out.println("call = " + url + ", message = " + message);
  }

  // 서비스 종료 시 호출하는 메서드
  public void disconnect() {
    System.out.println("closed url = " + url);
  }
}

```

{% endtab %} {% tab title=".java" %}

```java
public class BeanLifeCycleTest {

  @Test
  public void lifeCycleTest() {
    ConfigurableApplicationContext ac =
        new AnnotationConfigApplicationContext(NetworkClient.class);
    NetworkClient client = ac.getBean(NetworkClient.class);

    // ApplicatoinContext 대신, 이를 상속하는
    // AnnotationConfigApplicationContext이나
    // ConfigurableApplicationContext를 사용하면 close()를 제공한다.
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

![](../../.gitbook/assets/kimyounghan-spring-core-principle/08/screenshot%202021-04-13%20오전%208.36.14.png)

코드를 실행하면 url이 null로 나온다. 생성자를 만들 때 url 값을 넘겨준 게 없고, `setUrl()` 이후에 url 호출이 없었기 때문에 null이 나온다.

---

스프링 빈은 `객체 생성 -> 의존 관계 주입` 이라는 라이프 사이클을 가진다. 생성자 주입은 생성할 때 파라미터로 주입하므로 예외다.

스프링 빈은 객체를 생성하고 의존 관계 주입이 다 끝난 후에야 필요한 데이터를 사용할 수 있다. 객체 안에 필요한 값을 연결하고 진짜 처음으로 일을 시작하는 초기화 작업은 의존
관계 주입까지 모두 완료된 다음에 호출되어야 한다. 그런데 개발자가 의존 관계 주입이 모두 완료된 시점을 어떻게 알 수 있을까?

스프링은 의존 관계 주입이 완료되면, 스프링 빈에게 콜백 베서드로 초기화 시점을 알려준다. 스프링 컨테이너가 종료되기 전에 소멸 콜백도 보내준다. 따라서 안전하게 작업을 종료할 수
있다.

## 스프링 빈의 이벤트 라이프 사이클

1. 스프링 컨테이너 생성
2. 스프링 빈 생성
3. 의존 관계 주입
4. **초기화 콜백**
    - 빈이 생성되고 빈의 의존 관계 주입이 완료된 후 호출된다.
5. 사용
6. **소멸 전 콜백**
    - 빈이 소멸되기 직전에 호출된다.
7. 스프링 종료

## 객체의 생성과 초기화의 분리

그렇다면 그냥 생성자로 처리해버리면 귀찮은 과정을 거치지 않아도 되는 거 아닐까? 하지만 객체 생성과 초기화는 분리하는 것이 좋다. 단일 책임 원칙의 연장선이다.

생성자는 필수 정보(파라미터)를 받아 메모리에 할당해서 객체를 생성하는 책임을 가진다. 초기화는 이렇게 생성된 값들을 활용해 외부 커넥션을 연결하는 등 무거운 동작을 수행한다.

따라서 생성자 안에서 무거운 초기화 작업을 함께 하는 것 보다는, 생성과 초기화를 명확하게 나누는 것이 유지 보수 관점에서 좋다. 객체를 생성해놓고 실제 외부 커넥션을 맺는 작업은 최초에 정말 액션이 주어질 때까지 지연시킬 수 있는 장점도 있다.

물론 초기화 작업이 내부 값만 약간 변경하는 정도로 단순하다면 생성자에서 한 번에 다 처리하는 게 나을 수도 있다.

## 빈 생명 주기 콜백의 종류

- 인터페이스
   - InitializingBean
   - DisposableBean
- 설정 정보에 초기화 / 종료 메서드 지정
- @PostConstruct, @PreDestroy 애너테이션