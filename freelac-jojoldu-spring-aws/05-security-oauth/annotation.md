# 애너테이션 기반으로 개선하기

{% tabs %}
{% tab title="IndexController.java" %}
```java
@Controller
@RequiredArgsConstructor
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model) {
        ...

        SessionUser user = (SessionUser) httpSession.getAttribute("user");

        ...
    }
}
```
{% endtab %}
{% endtabs %}

`IndexController`는 위와 같이 세션 값을 불러오고 있다. 다른 컨트롤러와 메서드에서 세션 값이 필요하면 그때마다 직접 세션에서 값을 가져와야 하는 것이다. 따라서 이 부분을 애너테이션으로 간단하게 사용할 수 있도록 개선해보자.

{% tabs %}
{% tab title="IndexController.java" %}
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```
{% endtab %}
{% endtabs %}

### @Target(ElementType.PARAMETER)

해당 애너테이션이 생성될 수 있는 위치를 지정한다. `PARAMETER`로 지정하면 메서드의 파라미터로 선언도니 객체에서만 사용할 수 있다. 클래스 선언문에 쓰고 싶다면 `TYPE`으로 지정하면 된다.

### @interface

해당 파일을 애너테이션으로 설정한다. 즉, `LoginUser`라는 이름의 애너테이션이 생성되는 것이다.

{% tabs %}
{% tab title="LoginUserArgumentResolver.java" %}
```java
@RequiredArgsConstructor
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {
    
    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());
        
        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }
}
```
{% endtab %}
{% endtabs %}

### supportsParameter()

컨트롤러 메서드에서 특정 파라미터를 지원하는지 판단한다. 여기서는 파라미터에 `@LoginUser` 애너테이션이 붙어있고, 파라미터 클래스 타입이 `SessionUser`인 경우 true를 반환한다.

### resolverArgument()

파라미터에 전달할 객체를 생성한다. 여기서는 세션에서 객체를 가져오게 된다.

이제, `LoginUserArgumentResolver`가 스프링에서 인식되도록 `WebMvcConfigurer`에 추가하자.

{% tabs %}
{% tab title="WebConfig.java" %}
```java
@RequiredArgsConstructor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final LoginUserArgumentResolver loginUserArgumentResolver;
    
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(loginUserArgumentResolver);
    }
}
```
{% endtab %}
{% endtabs %}

`HandlerMethodArgumentResolver`는 항상 `WebMvcConfigurer`의 `addArgumentResolvers`를 통해 값을 추가해야 한다.

설정이 끝났으므로 `IndexController`에서 반복되는 부분을 `@LoginUser`로 변경한다.

{% tabs %}
{% tab title="After" %}
```java
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user) {
        model.addAttribute("posts", postsService.findAllDesc());

        if(user != null) {
            model.addAttribute("userName", user.getName());
        }

        return "index";
    }
}
```
{% endtab %}
{% tab title="Before" %}
```java
@Controller
@RequiredArgsConstructor
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());

        SessionUser user = (SessionUser) httpSession.getAttribute("user");

        if(user != null) {
            model.addAttribute("userName", user.getName());
        }

        return "index";
    }

    ...
}
```
{% endtab %}
{% endtabs %}

### @LoginUser SessionUser user

기존에 `httpSession.getAttribute()`로 가져오던 세션 정보 값을 애너테이션으로 대체했다. 이제 다른 컨트롤러에서도 세션 정보가 필요하면 `@LoginUser`를 재사용하면 된다.