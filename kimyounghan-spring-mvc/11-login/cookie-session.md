# 쿠키, 세션

로그인의 상태를 유지할 때는 쿠키를 사용한다. 정보를 쿼리 파라미터로 계속 보내는 건 어렵고 번거롭기 때문이다.

## 쿠키

로그인이 성공하면 서버에서 HTTP 응답에 쿠키를 담아 브라우저에 전달한다. 그럼 브라우저는 앞으로 그 쿠키를 계속 보낸다.

- 영속 쿠키
    - 만료 날짜를 입력하면 해당 날짜까지만 유지한다.
- 세션 쿠키
    - 만료 날짜를 생략하면 브라우저를 종료할 때까지만 유지한다.
    - 여기에서 말하는 세션은 HTTP 세션 등과는 관련 없다.

이 프로젝트에서는 브라우저를 종료하면 로그아웃 되어야 하므로 세션 쿠키가 필요하다.

### 로그인

{% tabs %} {% tab title="LoginController.java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class LoginController {

    ...

    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        ...

        // 시간 정보를 주지 않았으므로 세션 쿠키가 된다. 즉, 브라우저 종료 시 로그아웃 된다.
        Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
        // HTTP 응답에 쿠키를 넣어 보낸다.
        response.addCookie(idCookie);

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

    @GetMapping("/")
    /*
    HttpServletRequest 등 쿠키를 가져오는 방법은 많으나 스프링이 제공하는 @CookieValue를 사용해보자.
    로그인 안 한 사용자도 home에 접속할 수 있어야 하므로 required는 false로 둔다.
    들어오는 memberId는 String이지만 스프링이 Long으로 컨버팅해준다.
    */
    public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {
        if (memberId == null) {
            return "home";
        }

        Member loginMember = memberRepository.findById(memberId);
        if (loginMember == null) {
            return "home";
        }

        model.addAttribute("member", loginMember);
        return "loginHome";
    }
}
```

{% endtab %} {% endtabs %}

- @CookieValue
    - 쿠키 조회
- required = false
    - 로그인 하지 않은 사용자도 홈에 접근하게 한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오전%2011.13.50.png)

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오전%2011.14.34.png)

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오전%2011.16.48.png)

쿠키가 생성된 걸 확인할 수 있다.

### 로그아웃

{% tabs %} {% tab title="LoginController.java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class LoginController {

    ...

    @PostMapping("/logout")
    public String logout(HttpServletResponse response) {
        expireCookie(response, "memberId");
        return "redirect:/";
    }

    private void expireCookie(HttpServletResponse response, String cookieName) {
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0);
        response.addCookie(cookie);
    }
}
```

{% endtab %} {% endtabs %}

로그아웃 할 때 서버에서 쿠키의 종료 날짜를 0으로 한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오전%2011.30.42.png)

Max-Age가 0으로 변경되었다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오전%2011.30.57.png)

삭제된 것을 확인할 수 있다.