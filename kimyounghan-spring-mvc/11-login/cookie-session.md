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

## 보안 이슈

쿠키는 심각한 보안 이슈가 있다.

- 쿠키 값을 임의로 변경할 수 있다.
    - 클라이언트가 쿠키를 강제로 변경하면 다른 사용자가 된다.
    - `개발자 모드 - Application - Cookie`에서 변경할 수 있다.
        - `memberId=1` 에서 `memberId=2`로 바꾸면 다른 사용자가 보인다.
- 쿠키 정보를 훔쳐갈 수 있다.
    - 중요한 정보가 브라우저에 저장될 뿐 아니라 네트워크 요청마다 서버로 전달된다.
        - PC나 네트워크 전송 구간에서 털릴 수 있다.
- 해커가 쿠키를 한 번 훔쳐가변 평생 사용할 수 있다.
    - 계속 악의적인 요청을 시도할 수 있다.

### 대안

- 쿠키에 중요한 값을 노출하지 않는다.
- 사용자 별로 예측 불가능한 랜덤 토큰을 노출한다.
    - 서버에서는 토큰과 사용자 ID를 매핑해서 인식한다.
    - 토큰을 서버에서 관리한다.
    - 해커가 임의로 값을 넣어도 찾을 수 없도록 예측 불가능해야 한다.
- 토큰을 털어도 시간이 지나면 사용할 수 없도록 토큰 만료시간을 30분 등으로 짧게 유지한다.
    - 해킹이 의심되면 서버에서 해당 토큰을 강제로 제거하면 된다.