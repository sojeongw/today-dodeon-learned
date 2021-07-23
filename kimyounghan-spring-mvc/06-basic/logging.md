# 로깅

- 운영에서는 System.out.println 대신 별도의 로깅 라이브러리를 사용한다.
- 스프링 부트를 사용하면 spring-boot-starter-logging 라이브러리가 포함된다.
- 스프링 부트 로깅 라이브러리가 기본으로 사용하는 라이브러리
    - slf4j
        - logback, log4j 등 다양한 라이브러리를 통합해 인터페이스로 제공하는 라이브러리
    - logback
        - 실제 로깅 구현체

## 로그 선언

```java
// import 할 때 인터페이스인 slf4j인지 꼭 확인한다.

import org.slf4j.Logger;

@RestController
public class LogTestController {
    private final Logger log = LoggerFactory.getLogger(getClass());
}

```

```java
import org.slf4j.Logger;

@RestController
public class LogTestController {
    // 사용하는 클래스를 직접 지정해줄 수도 있다.
    private final Logger log = LoggerFactory.getLogger(LogTestController.class);
}

```

```java

@Slf4j
class Example {

}
```

- 코드나 롬복으로 사용할 수 있다.

{% tabs %} {% tab title="LogTestController.java" %}

```java
// Controller라고 하면 View 이름을 반환하고
// Restcontroller로 하면 "ok"라는 스트링이 그대로 반환된다.
@RestController
public class LogTestController {
    private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String logTest() {
        String name = "Spring";

        System.out.println("name = " + name);

        log.trace("trace log={}", name);
        log.debug("debug log={}", name);
        log.info("info log={}", name);
        log.warn("warn log={}", name);
        log.error("error log = {}", name);

        return "ok";
    }
}
```

{% endtab %} {% endtabs %}

### 매핑 정보

- @Controller
    - 반환값이 String이면 뷰 이름으로 인식된다.
    - 뷰를 찾고 뷰가 렌더링 된다.
- @RestController
    - HTTP 메시지 바디에 바로 입력한다.
    - @ResponseBody와 관련있다.

### 로그 포맷

- 시간
- 로그 레벨
- 프로세스 ID
- 스레드 명
- 클래스 명
- 로그 메시지

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-02-28%20오후%207.33.41.png)

- log로 찍은 데이터는 시간, 스레드, 컨트롤러 등 자세한 정보와 함께 출력된다.
- 기본 설정에서는 info, warn, error만 출력된다.

### 로그 레벨

- trace > debug > info > warn > error
- 보통 개발 서버는 debug, 운영 서버는 info로 출력한다.
- 기본값은 info 레벨로 되어있다.
    - warn, error가 포함된다.

{% tabs %} {% tab title="application.properties" %}

```properties
# 전체 로그 레벨 설정
# 오만가지 로그가 다 쌓이게 된다.
logging.level.root=info
# hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=trace
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-02-28%20오후%207.35.41.png)

- properties에 trace로 설정을 해주면 trace와 debug도 출력된다.
- trace는 빼고 debug만 보고 싶으면 properties에 debug로 설정한다.

## 올바른 로그 사용법

```java
class Example {
    public void example() {
        // 잘못된 예
        log.debug("data = " + data);
        // 올바른 예
        log.debug("data = {}", data);
    }
}
```

- +로 하면 더하기 연산이 실행된다.
    - 심지어 log.debug이면 debug 레벨이 아니더라도 일단 연산을 하고 본다.
- {}로 하면 그런 일이 발생하지 않는다.

## log 사용의 장점

- 스레드 정보, 클래스 이름 같은 부가 정보를 볼 수 있다.
- 출력 모양을 조정할 수 있다.
- 환경에 따라 로그 레벨을 조절할 수 있다.
- 로그 레벨을 코드가 아니라 설정으로 변경할 수 있다.
- System.out은 콘솔에만 남지만 log는 파일, 네트워크 등 별도의 위치에 로그를 남길 수 있다.
    - 파일의 경우 날짜, 용량에 따라 로그를 분할할 수도 있다.
- 성능도 System.out보다 좋다.
    - 내부 버퍼링, 멀티 스레드 등 성능 최적화가 다 되어있어 로그가 한 번에 많이 와도 잘 해결해준다.
    - 실무에서는 꼭 log를 사용해야 한다.

### Reference

[스프링 부트가 제공하는 로그 기능](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging)