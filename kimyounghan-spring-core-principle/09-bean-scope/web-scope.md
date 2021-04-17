# 웹 스코프

웹 환경에서만 동작하는 스코프다. 프로토타입과 다르게 스프링이 해당 스코프의 종료 시점까지 관리한다. 따라서 종료 메서드가 호출된다.

## 종류

### request

- HTTP 요청 하나가 들어오고 나갈 때까지 유지되는 스코프
- 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-17%20오후%201.31.44.png)

0.00001초 차이로 동시에 요청해도 클라이언트 A와 B는 각각 다른 스프링 빈이 생성된다.

### session

- HTTP Session과 동일한 생명 주기를 가지는 스코프

### application

- 서블릿 컨텍스트와 동일한 생명주기를 가지는 스코프

### websocket

- 웹 소켓과 동일한 생명 주기를 가지는 스코프

---

이 챕터에서는 request 스코프만 예제로 설명한다. 나머지는 범위만 다르고 동작 방식이 비슷하기 때문이다.

```groovy
// web 라이브러리 추가
implementation 'org.springframework.boot:spring-boot-starter-web'
```

웹 스코프는 웹 환경에서만 동작하므로 위의 라이브러리를 추가하자.

```text
Tomcat started on port(s): 8080 (http) with context path ''
Started CoreApplication in 0.914 seconds (JVM running for 1.528)
```

그럼 이렇게 톰캣이 띄워지게 된다. 이렇듯 `spring-boot-starter-web` 라이브러리를 추가하면 스프링 부트가 내장 톰캣 서버를 활용해 웹 서버와 함께 실행한다.

```properties
server.port=9090
```

만약 기본 포트인 8080을 다른 곳에서 사용중이라면 `application.properties`에서 포트를 변경한다.

스프링 부트는 웹 라이브러리가 없으면 지금까지 학습한 `AnnotationConfigApplicationContext`를 기반으로 애플리케이션을 구동한다.

반면, 웹 라이브러리가 추가되면 웹과 관련된 추가 설정과 환경들이 필요해 `AnnotationConfigServletWebServerApplicationContext`를 기반으로
구동한다.

## 예제

만약 동시에 여러 HTTP 요청이 오면, 정확히 어떤 요청이 남긴 로그인지 구분하기 어렵다. 이럴 때 사용하기 좋은 것이 request 스코프다.

```text
[d06b992f...] request scope bean create
[d06b992f...] [http://localhost:8080/log-demo] controller test
[d06b992f...] [http://localhost:8080/log-demo] service id = testId
[d06b992f...] request scope bean close
```

`UUID + 요청URL + 메시지` 형태로 출력할 것이다. UUID로 HTTP 요청을 구분하고 어떤 URL을 요청해서 남은 로그인지 확인한다.

{% tabs %} {% tab title="MyLogger.java" %}

```java

@Component
@Scope(value = "request")
public class MyLogger {

  private String uuid;
  private String requestURL;

  public void setRequestURL(String requestURL) {
    this.requestURL = requestURL;
  }

  public void log(String message) {
    System.out.println("[" + uuid + "]" + "[" + requestURL + "] " + message);
  }

  @PostConstruct
  public void init() {
    // 유니크한 아이디 생성
    uuid = UUID.randomUUID().toString();
    System.out.println("[" + uuid + "] request scope bean created: " + this);
  }

  @PreDestroy
  public void close() {
    System.out.println("[" + uuid + "] request scope bean closed: " + this);
  }
}
```

{% endtab %} {% endtabs %}

request 스코프이기 때문에 HTTP 요청마다 빈이 생성되고 소멸될 것이다. 빈이 생성되는 시점에는 `@PostConstruct`로 uuid를 생성해 저장해둔다. 빈이 요청 당
하나씩 생성되므로 uuid를 저장해두면 다른 요청과 구분할 수 있다.

빈이 소멸되는 시점에는 `@PreDestroy`를 사용해 종료 메시지를 남긴다. `requestURL`은 빈이 생성되는 시점에는 알 수 없어 외부에서 setter로 입력받는다.

{% tabs %} {% tab title="LogDemoController.java" %}

```java

@Controller
// 생성자에 자동으로 autowired로 자동 주입된다.
@RequiredArgsConstructor
public class LogDemoController {

  private final LogDemoService logDemoService;
  private final MyLogger myLogger;

  @RequestMapping("log-demo")
  @ResponseBody
  public String logDemo(HttpServletRequest request) {
    // 로그에 찍기 위해 URL을 받아온다.
    String requestURL = request.getRequestURL().toString();
    myLogger.setRequestURL(requestURL);

    myLogger.log("controller test");
    logDemoService.logic("testId");

    return "OK";
  }
}
```

{% endtab %} {% endtabs %}

로거를 테스트하기 위한 컨트롤러를 만든다. `reqeustURL`을 `myLogger`에 저장해두면 `myLogger`는 HTTP 요청 당 구분되므로 값이 섞이지 않게 된다.

참고로 이렇게 저장하는 부분은 컨트롤러 보다는 공통 처리가 가능한 스프링 인터셉터나 서블릿 필터를 이용하는 게 좋다. 여기서는 예제를 단순화하기 위해 이렇게 구현했다.

{% tabs %} {% tab title="LogDemoService.java" %}

```java

@Service
@RequiredArgsConstructor
public class LogDemoService {

  private final MyLogger myLogger;

  public void logic(String id) {
    myLogger.log("service id = " + id);
  }
}
```

{% endtab %} {% endtabs %}

서비스 계층에서도 로그를 찍어보자. request 스코프를 사용하지 않고 모든 정보를 서비스 계층에 넘긴다면 파라미터가 많아서 지저분해진다. `requestURL`처럼 웹과 관련된
정보가 웹과 관련없는 서비스 계층까지 넘어가는 문제도 있다. 웹과 관련된 부분은 컨트롤러까지만 사용하고 서비스 계층은 웹 기술에 종속되지 않아야 하므로 가급적 순수하게 유지하는
것이 유지 보수에 좋다.

reqeust 스코프의 `MyLogger` 덕분에 웹과 관련된 부분을 파라미터로 넘기지 않고 `MyLogger`의 멤버 변수에 저장해 코드와 계층을 깔끔하게 유지할 수 있다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-17%20오후%203.02.25.png)

실행해보면 위와 같은 에러가 뜬다. `LogDemoController`가 만들어지려면 `MyLogger`를 주입받아야 하는데 `MyLogger`는 실제 HTTP 요청이 있을 때
빈을 생성하므로 실패한 것이다.

## 스코프와 Provider

첫번째 해결 방안은 Provider를 사용하는 것이다.

{% tabs %} {% tab title="LogDemoController.java" %}

```java

@Controller
@RequiredArgsConstructor
public class LogDemoController {

  private final LogDemoService logDemoService;
  // myLogger를 찾을 수 있는, DL할 수 있는 객체가 주입되어 주입 시점에 myLogger을 주입받을 수 있다.
  private final ObjectProvider<MyLogger> myLoggerProvider;

  @RequestMapping("log-demo")
  @ResponseBody
  public String logDemo(HttpServletRequest request) {
    // DL로 원하는 객체를 가져온다.
    MyLogger myLogger = myLoggerProvider.getObject();

    String requestURL = request.getRequestURL().toString();
    myLogger.setRequestURL(requestURL);

    myLogger.log("controller test");
    logDemoService.logic("testId");

    return "OK";
  }
}
```

{% endtab %} {% tab title="LogDemoService.java" %}

```java

@Service
@RequiredArgsConstructor
public class LogDemoService {

  private final ObjectProvider<MyLogger> myLoggerProvider;

  public void logic(String id) {
    MyLogger myLogger = myLoggerProvider.getObject();
    myLogger.log("service id = " + id);
  }
}
```

{% endtab %} {% endtabs %}

`ObjectProvider` 덕분에 `ObjectProvider.getObject()`를 호출할 때까지 request 스코프 빈을 생성해달라고 스프링 컨테이너에 요청하는 시점을
지연할 수 있다.

`ObjectProvider.getObject()`를 호출하는 시점에는 HTTP 요청이 진행 중이므로 request 스코프 빈이 생성된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-17%20오후%203.18.12.png)

이제 `http://localhost:8080/log-demo`에 접속하면 위와 같은 결과가 뜬다. 같은 요청이면 컨트롤러와 서비스 모두 같은 uuid가 출력된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-17%20오후%203.19.15.png)

한 번 더 요청하면 다른 uuid로 로그가 찍힌다.

## 스코프와 프록시

두 번째 해결 방안은 프록시 방식이다.

{% tabs %} {% tab title="MyLogger.java" %}

```java

@Component
// 프록시 모드 추가
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
  ...
}
```

{% endtab %} {% tab title="LogDemoController.java" %}

```java

@Controller
@RequiredArgsConstructor
public class LogDemoController {

  private final LogDemoService logDemoService;
  // 원복한다.
  private final MyLogger myLogger;

  @RequestMapping("log-demo")
  @ResponseBody
  public String logDemo(HttpServletRequest request) {
    String requestURL = request.getRequestURL().toString();
    myLogger.setRequestURL(requestURL);

    myLogger.log("controller test");
    logDemoService.logic("testId");

    return "OK";
  }
}
```

{% endtab %} {% tab title="LogDemoService.java" %}

```java

@Service
@RequiredArgsConstructor
public class LogDemoService {

  // 원복한다.
  private final MyLogger myLogger;

  public void logic(String id) {
    myLogger.log("service id = " + id);
  }
}
```

{% endtab %} {% endtabs %}

`proxyMode`를 추가하면 `MyLogger`의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입한다.

적용 대상이 클래스면 `TARGET_CLASS`, 인터페이스면 `INTERFACES`를 선택한다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-17%20오후%203.34.59.png)

`Provider` 사용 시와 같이 정상적으로 동작한다.

### 동작 원리

- CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어 주입한다.
- 이 가짜 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직을 가진다.
- 가짜 프록시 객체는 실제 request 스코프와는 관계가 없다. 어디든 사용할 수 있다.
  - 그냥 가짜이고, 내부에 단순한 위임 로직만 있다.
  - `myLogger`처럼 사용하는 객체가 간단하게 싱글턴처럼 동작한다.

{% tabs %} {% tab title=".java" %}

```java

@Controller
@RequiredArgsConstructor
public class LogDemoController {

  private final LogDemoService logDemoService;
  private final MyLogger myLogger;

  @RequestMapping("log-demo")
  @ResponseBody
  public String logDemo(HttpServletRequest request) {
    System.out.println("myLogger = " + myLogger.getClass());

    ...

    return "OK";
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-17%20오후%203.39.36.png)

`myLogger`를 찍어보면 이상한 값이 나왔다.

`@Scope`의 `proxyMode = ScopedProxyMode.TARGET_CLASS`를 설정하면 스프링 컨테이너는 `CGLIB`이라는 바이트 코드 조작
라이브러리로 `MyLogger`를 상속 받은 가짜 프록시 객체를 생성한다.

일단 스프링 컨테이너에 `myLogger`라는 이름의 가짜 프록시 객체를 넣어두고 실제 필요한 시점에 가져와서 동작하는
것이다. `ac.Bean("myLoger", MyLogger.class)`로 찍어봐도 프록시 객체가 조회된다. 그래서 의존 관계 주입도 이 가짜 프록시 객체로 주입된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/09/screenshot%202021-04-17%20오후%203.44.02.png)

가짜 프록시 객체는 요청이 오면 그때가 돼서야 내부에서 진짜 빈을 요청하는 위임 로직을 실행한다.

1. 가짜 프록시 객체는 내부에 진짜 `myLogger`를 찾는 방법을 알고 있다.
2. 클라이언트가 `myLogger.logic()`을 호출하면 가짜 프록시 객체의 메서드가 호출된다.
3. 가짜 프록시 객체는 request 스코프의 진짜 `myLogger.logic()`을 호출한다.
4. 가짜 프록시 객체는 원본 클래스를 상속받아 만들어졌기 때문에 사용하는 클라이언트 객체는 원본이든 아니든 다형성을 활용해 동일하게 사용한다.

## 정리

- 프록시 객체 덕분에 클라이언트는 싱글턴 빈을 사용하듯 편리하게 request 스코프를 사용할 수 있다.
- Provider와 프록시의 핵심은 진짜 객체 조회를 꼭 필요한 시점까지 지연 처리 한다는 점이다.
- 애너테이션 설정 변경만으로 원본 객체를 프록시 객체로 대체할 수 있다.
  - 다형성과 DI가 가진 큰 강점이다.
- 꼭 웹 스코프가 아니어도 프록시는 사용할 수 있다.

## 주의점

- 싱글턴을 사용하는 것 같지만 다르게 동작하므로 주의해서 사용해야 한다.
- 이런 특별한 scope는 꼭 필요한 곳에만 최소로 사용하자. 유지보수가 어려워질 수 있다.