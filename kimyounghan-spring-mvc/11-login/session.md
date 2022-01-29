# 세션

- 쿠키의 보안 이슈를 해결하려면 중요한 정보를 모두 서버에 저장해야 한다.
- 클라이언트와 서버는 추정할 수 없는 임의의 식별자로 연결되어야 한다.

이런 방식을 세션이라고 한다.

## 동작 방식

### 로그인

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오전%2011.53.30.png)

- 아이디와 패스워드를 전달하면 서버에서 확인한다.

### 세션 생성

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오전%2011.53.38.png)

```text
Cookie: mySessionId=zz0101xx-bab9-4b92-9b32-dadb280f4b61
```

- 회원이 맞다면 임의의 값으로 세션 ID를 생성한다.
    - UUID는 추정이 불가능하다.
- 생성한 세션 ID와 세션에 보관할 값을 서버의 세션 저장소에 보관한다.

### 응답

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오전%2011.53.47.png)

- 클라이언트와 서버를 연결하려면 결국 쿠키를 쓰긴 해야 한다.
- 서버
    - 클라이언트에 mySessionId라는 이름으로 세션 ID만 쿠키에 담아 전달한다.
- 클라이언트
    - 쿠키 저장소에 mySessionId 쿠키를 보관한다.

중요한 포인트는 회원 정보가 전혀 클라이언트에 전달되지 않는다는 것이다. 오직 추정 불가한 세션 ID만 쿠키로 넘겨준다.

### 요청

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오전%2011.53.55.png)

- 클라이언트
    - 요청할 때마다 mySessionId 쿠키를 전달한다.
- 서버
    - 클라이언트가 보낸 mySessionId 쿠키 정보로 세션 저장소를 조회한다.
    - 로그인할 때 보관했던 세션 정보를 사용한다.

## 정리

보안 문제를 해결하기 위해, 세션을 이용해 서버에서 중요 정보를 관리한다.

- 쿠키 값을 변조할 수 있다.
    - 예상 불가능한 복잡한 세션 ID를 사용한다.
- 쿠키 정보가 클라이언트 해킹으로 털릴 수 있다.
    - 세션 ID가 털려도 중요 정보가 없다.
- 쿠키를 탈취해 영원히 사용한다.
    - 토큰을 털어도 세션 만료 시간을 짧게 해서 사용할 수 없게 한다.
    - 해킹이 의심되면 서버에서 해당 세션을 강제로 제거한다.

## 직접 만든 세션 적용하기

- 세션 생성
    - sessionId 생성
        - 임의의 추정 불가능한 랜덤 값
    - 세션 저장소에 sessionId와 보관할 값을 저장한다.
    - sessionId로 응답 쿠키를 생성해 클라이언트에 전달한다.
- 세션 조회
    - 클라이언트가 요청한 sessionId 쿠키 값으로 세션 저장소에 보관한 값을 조회한다.
- 세션 만료
    - 클라이언트가 요청한 sessionId 쿠키 값으로 세션 저장소에 보관한 sessionId와 값을 제거한다.

{% tabs %} {% tab title="SessionManager.java" %}

```java

@Component
public class SessionManager {

    private static final String SESSION_COOKIE_NAME = "mySessionId";
    private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

    public void createSession(Object value, HttpServletResponse response) {
        // 세션 ID를 생성한 뒤 값을 세션에 저장한다.
        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        // 쿠키를 생성한다.
        Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(mySessionCookie);
    }

    public Object getSession(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);

        if (sessionCookie == null) {
            return null;
        }

        return sessionStore.get(sessionCookie.getValue());
    }

    public void expire(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);

        if (sessionCookie != null) {
            sessionStore.remove(sessionCookie.getValue());
        }
    }

    private Cookie findCookie(HttpServletRequest request, String cookieName) {
        if (request.getCookies() == null) {
            return null;
        }

        return Arrays.stream(request.getCookies())
                .filter(c -> c.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }
}
```

{% endtab %} {% tab title="SessionManagerTest.java" %}

```java
class SessionManagerTest {

    SessionManager sessionManager = new SessionManager();

    @Test
    void sessionTest() {
        // 세션 생성
        MockHttpServletResponse response = new MockHttpServletResponse();
        Member member = new Member();
        sessionManager.createSession(member, response);

        // 응답에 있던 쿠키를 요청에 저장
        MockHttpServletRequest request = new MockHttpServletRequest();
        request.setCookies(response.getCookies());

        // 세션 조회
        Object result = sessionManager.getSession(request);
        assertThat(result).isEqualTo(member);

        // 세션 만료
        sessionManager.expire(request);
        Object expired = sessionManager.getSession(request);
        assertThat(expired).isEqualTo(null);
    }
}
```

{% endtab %} {% endtabs %}

세션을 관리하는 빈을 만든다.

{% tabs %} {% tab title="LoginController.java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class LoginController {

    private final SessionManager sessionManager;

    @PostMapping("/login")
    public String loginV2(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        ...

        sessionManager.createSession(loginMember, response);

        return "redirect:/";
    }

    @PostMapping("/logout")
    public String logoutV2(HttpServletRequest request) {
        sessionManager.expire(request);
        return "redirect:/";
    }
}
```

{% endtab %} {% tab title="HomeController.java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class HomeController {

    private final SessionManager sessionManager;

    @GetMapping("/")
    public String homeLoginV2(HttpServletRequest request, Model model) {
        // 세션 저장소에서 정보를 가져온다.
        Member member = (Member) sessionManager.getSession(request);

        if (member == null) {
            return "home";
        }

        model.addAttribute("member", member);
        return "loginHome";
    }
}
```

{% endtab %} {% endtabs %}

수동으로 쿠키를 생성하고 만료시키던 로직 대신 sessionManager를 주입받아 사용한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오후%201.39.21.png)

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오후%201.39.34.png)

쿠키 값이 mySessionId로 바뀌었다.