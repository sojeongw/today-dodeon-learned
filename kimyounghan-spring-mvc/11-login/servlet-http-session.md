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