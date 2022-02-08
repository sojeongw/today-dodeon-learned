# ArgumentResolver 활용

ArgumentResolver를 이용해 세션 확인을 좀 더 간단하게 해보자.

{% tabs %} {% tab title="Login.java" %}

```java

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Login {
}
```

{% endtab %} {% endtabs %}

- @Target(ElementType.PARAMETER)
    - 파라미터에서만 사용할 수 있게 설정
- @Retention(RetentionPolicy.RUNTIME)
    - 리플렉션 등을 활용할 수 있게 런타임 내내 애너테이션 정보가 남아있어야 할 때 사용

{% tabs %} {% tab title="HomeController.java" %}

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class HomeController {

    @GetMapping("/")
    public String homeLoginV3ArgumentResolver(@Login Member loginMember, Model model) {
        if (loginMember == null) {
            return "home";
        }

        model.addAttribute("member", loginMember);
        return "loginHome";
    }
}
```

{% endtab %} {% endtabs %}

- 만든 @Login 애너테이션을 붙여주면 ArgumentResolver가 동작한다.
    - 자동으로 세션에 있는 로그인 회원을 찾아준다.

{% tabs %} {% tab title="LoginMemberArgumentResolver.java" %}

```java

@Slf4j
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        log.info("supportsParameter 실행");

        // 파라미터 정보에서 @Login 애너테이션이 있는지 확인한다.
        boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
        // 파라미터가 Member 타입인지 확인한다.
        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());

        // 두 조건을 모두 만족하면 true가 반환되면서 resolveArgument()가 실행된다.
        return hasLoginAnnotation && hasMemberType;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) throws Exception {
        log.info("resolveArgument 실행");

        // HttpServletRequest을 NativeWebRequest에서 뽑는다.
        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
        HttpSession session = request.getSession(false);

        if (session == null) {
            return null;
        }

        return session.getAttribute(SessionConst.LOGIN_MEMBER);
    }
}
```

{% endtab %} {% endtabs %}

- supportsParameter()
    - @Login 애너테이션이 있으면서 Member 타입이면 해당 ArgumentResolver가 사용된다.
- resolveArgument()
    - 컨트롤러 호출 직전에 호출되어 필요한 파라미터 정보를 생성한다.
    - 즉, 세션에 있는 로그인 회원 정보인 Member 객체를 찾아 반환한다.
    - 이후 스프링 MVC는 컨트롤러의 homeLoginV3ArgumentResolver()를 호출하면서 여기서 반환된 Member 객체를 파라미터에 전달해준다.

{% tabs %} {% tab title="WebConfig.java" %}

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new LoginMemberArgumentResolver());
    }
}
```

{% endtab %} {% endtabs %}

- 항상 등록해주는 것을 잊지 말자.

![](../../.gitbook/assets/kimyounghan-spring-mvc/11/screenshot%202022-03-13%20오후%206.21.16.png)

참고로 여러 번 실행할 때는 캐시가 있어서 supportsParameter()는 한 번만 실행한다.

이렇게 ArgumentResolver를 활용하면 공통 작업이 필요할 때 컨트롤러를 더 편리하게 사용할 수 있다.