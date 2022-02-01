# 서블릿 HTTP 세션

서블릿도 우리가 직접 만든 SessionManager와 같은 방식의 HTTP 세션을 제공한다.

```text
Cookie: JSESSIONID=5B78E23B513F50164D6FDD8C97B0AD05
```

서블릿으로 HttpSession을 생성하면 JSESSIONID라는 이름의 쿠키를 생성하고 추정 불가능한 랜덤 값을 값으로 가진다.

## HttpSession

### 생성 및 조회

{% tabs %} {% tab title="SessionConst.java" %}

```java
// 상수만 가져다 쓰기 때문에 객체를 만들지 못하게 abstract나 interface로 만든다.
public abstract class SessionConst {
    public static final String LOGIN_MEMBER = "loginMember";
}
```

{% endtab %} {% tab title=".java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class LoginController {

    @PostMapping("/login")
    public String loginV3(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletRequest request) {
        ...

        // 세션이 있으면 반환하고 없으면 신규로 생성한다.
        HttpSession session = request.getSession();
        // 세션에 로그인 정보를 보관한다.
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);


        return "redirect:/";
    }
}
```

{% endtab %} {% endtabs %}

- 세션 생성
    - request.getSession(true)
        - 기본값이 true이므로 생략할 수 있다.
        - 세션이 있으면 기존 세션을 반환한다.
        - 없으면 새로 생성해서 반환한다.
    - request.getSession(false)
        - 세션이 있으면 기존 세션을 반환한다.
        - 없으면 생성하지 않고 null을 반환한다.

### 데이터 저장

session.setAttribute()로 하나의 세션에 여러 값을 저장할 수 있다.

{% tabs %} {% tab title=".java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class LoginController {

    @PostMapping("/logout")
    public String logoutV3(HttpServletRequest request) {
        // true면 세션을 만들어버리므로 어차피 없앨 것이기 때문에 false로 한다.
        HttpSession session = request.getSession(false);

        if (session != null) {
            session.invalidate();
        }

        return "redirect:/";
    }
}
```

{% endtab %} {% tab title=".java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class HomeController {

    @GetMapping("/")
    public String homeLoginV3(HttpServletRequest request, Model model) {
        // 로그인 하지 않은 사용자도 있으므로 false로 만든다.
        HttpSession session = request.getSession(false);

        if (session == null) {
            return "home";
        }

        Member loginMember = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);

        // 세션에 회원 데이터가 없으면 home으로 이동한다.
        if (loginMember == null) {
            return "home";
        }

        // 세션이 유지되면 로그인으로 이동한다.
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오후%202.11.30.png)

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오후%202.12.48.png)

JSESSIONID를 확인할 수 있다.

## @SessionAttribute

{% tabs %} {% tab title="HomeController.java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class HomeController {

    @GetMapping("/")
    public String homeLoginV3Spring(@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember, Model model) {
        // 세션에 회원 데이터가 없으면 home으로 이동한다.
        if (loginMember == null) {
            return "home";
        }

        // 세션이 유지되면 로그인으로 이동한다.
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
}
```

{% endtab %} {% endtabs %}

getAttribute()를 일일이 하지 않아도 알아서 데이터를 처리해준다.

## TrackingModes

```text
http://localhost:8080/;jsessionid=F59911518B921DF62D09F0DF8F83F872
```

로그인을 처음 시도하면 url에 jsessionid가 붙는다.

- 웹 브라우저가 쿠키를 지원하지 않을 때 url로 세션을 유지하는 방법
- url에 값을 계속 포함해서 전달해야 한다.
    - 번거로운 작업이라 거의 사용하지 않는다.
- 서버 입장에서는 쿠키 지원 여부를 최초에 판단하지 못하기 때문에 쿠키와 url의 jsessionid 모두 전달한다.

{% tabs %} {% tab title="application.properties" %}

```properties
server.servlet.session.tracking-modes=cookie
```

{% endtab %} {% endtabs %}

이렇게 설정하면 url에 더 이상 노출되지 않는다.

## 세션 정보

```java

@Slf4j
@RestController
public class SessionInfoController {

    @GetMapping("/session-info")
    public String sessionInfo(HttpServletRequest request) {
        HttpSession session = request.getSession(false);

        if (session == null) {
            return "세션이 없습니다.";
        }

        // 세션 데이터 출력
        session.getAttributeNames().asIterator()
                .forEachRemaining(name -> log.info("session name={}, value={}", name, session.getAttribute(name)));

        log.info("sessionId={}", session.getId());
        log.info("maxInactiveInterval={}", session.getMaxInactiveInterval());
        log.info("creationTime={}", new Date(session.getCreationTime()));
        log.info("lastAccessedTime={}", new Date(session.getLastAccessedTime()));
        log.info("isNew={}", session.isNew());

        return "세션 출력";
    }
}
```

```text
login? Member(id=1, loginId=test, name=테스터, password=test!)

session name=loginMember, value=Member(id=1, loginId=test, name=테스터, password=test!)
sessionId=6486F85F291FB8ED8AD5A1F388D780DA
maxInactiveInterval=1800
creationTime=Sun Mar 13 14:44:27 KST 2022
lastAccessedTime=Sun Mar 13 14:44:27 KST 2022
isNew=false
```

- sessionId
    - 세션 ID, JSESSIONID의 값
    - ex. 34B14F008AA3527C9F8ED620EFD7A4E1
- maxInactiveInterval
    - 세션의 유효 시간
    - ex. 1800초, 30분
- creationTime
    - 세션 생성 일시
- lastAccessedTime
    - 세션과 연결된 사용자가 최근에 서버에 접근한 시간
    - 클라이언트에서 서버로 sessionId(JSESSIONID)를 요청한 경우 갱신된다.
- isNew
    - 새로 생성된 세션인지 과거에 만들어져 클라이언트에서 서버로 sessionId(JSESSIONID)를 요청해서 조회된 세션인지 여부

## 타임아웃

- 세션은 사용자가 로그아웃을 직접 호출해서 session.invalidate()이 호출되는 경우에 삭제된다.
- 대부분의 사용자는 로그아웃 대신 웹 브라우저를 종료한다.
    - HTTP는 비연결성이기 때문에 서버는 사용자가 웹 브라우저를 종료했는지 알 수 없다.
    - 따라서 서버에서 언제 세션 데이터를 삭제해야 하는지 판단하기 어렵다.

이 경우 남아있는 세션을 계속 보관하면 문제가 발생할 수 있다.

- 세션과 관련된 쿠키(JSESSIONID)를 탈취 당하면 시간이 지나도 이 쿠키로 악의적인 요청을 보낼 수 있다.
- 세션은 기본적으로 메모리에 생성되기 때문에 OOM이 발생할 수도 있다.
    - 10만명이 로그인하면 세션도 10만개 생긴다.

### 종료 시점

- 세션 생성 시점을 기준으로 제한 시간을 잡으면 사이트를 이용하면서 30분마다 로그인 해야한다.
    - 제일 최근에 서버에 요청한 시간을 기준으로 시간을 유지하면 된다.

### 설정

{% tabs %} {% tab title="application.properties" %}

```properties
# 60초로 설정한다. 
server.servlet.session.timeout=60
```

{% endtab %} {% endtabs %}

- 기본값은 30분(1800초)
- 글로벌로 설정할 때는 60분, 120분 처럼 분 단위로 설정하는 게 좋다.
- 60초보다 작은 값은 설정할 수 없다.

```java
session.setMaxInactiveInterval(1800);
```

- 특정 세션의 시간을 설정한다면 setMaxInactiveInterval()을 이용한다.

### 타임아웃 발생

- 해당 세션에 JSESSIONID를 전달하는 HTTP 요청이 있으면 현재 시간으로 다시 초기화 한다.
    - 설정한 시간만큼 세션을 추가로 사용할 수 있다.
- LastAccessedTime으로부터 timeout 만큼의 시간이 지나면 WAS가 세션을 제거한다.

### 정리

서블릿의 HttpSession의 타임아웃 기능 덕분에 세션을 안전하고 편리하게 사용할 수 있다.

- 실무에서는 세션에 최소한의 데이터만 보관해야 한다.
    - 보관한 데이터 용량 * 사용자 수로 세션의 메모리 사용량이 급격하게 늘어나 장애로 번질 수 있다.
- 세션 시간을 너무 길게 가져가면 메모리 사용이 누적되므로 적당한 시간을 선택한다.
    - 기본이 30분이니 이걸 기준으로 고민한다.