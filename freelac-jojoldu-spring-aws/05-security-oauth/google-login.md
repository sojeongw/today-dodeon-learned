# 구글 로그인 연동하기

## 구글 서비스 등록

`src/main/resources`에 `application-oauth.properties`를 생성한다.

{% tabs %}
{% tab title="application-oauth.properties" %}
```properties
spring.security.oauth2.client.registration.google.client-id=클라이언트 ID
spring.security.oauth2.client.registration.google.client-secret=클라이언트 보안 비밀번호
spring.security.oauth2.client.registration.google.scope=profile, email
```
{% endtab %}
{% endtabs %}

`scope`를 profile과 email로 한정한 이유는 기본값이 openid, profile, email이기 때문이다. openid라는 scope가 있으면 Open Id Provider로 인식해 여기에 해당하는 구글과 그렇지 않은 네이버, 카카오에 대해 각각 `OAuth2Service`를 만들어줘야 한다. 즉, 하나로 사용하기 위해 일부러 openid scope를 제외하고 등록한다.

### properties

스프링 부트에서는 `application-xxx.properties`로 만들면 `xxx`라는 이름의 profile을 생성해 관리할 수 있다. `profile=xxx`처럼 호출하면 해당 properties의 설정을 가져올 수 있는 것이다.

{% tabs %}
{% tab title="application.properties" %}
```properties
spring.profiles.include=oauth
```
{% endtab %}
{% endtabs %}

이렇게 추가하면 이제부터 이 설정값을 사용할 수 있게 된다.

## gitignore 등록

클라이언트 ID와 보안 비밀번호는 보안이 중요한 정보다. 따라서 깃헙에 올라가지 않도록 아래의 코드를 `.gitignore`에 추가한다.

{% tabs %}
{% tab title=".gitignore" %}
```gitignore
application-oauth.properties
```
{% endtab %}
{% endtabs %}

만약 동작하지 않는다면 캐시문제이므로 아래의 명령어로 캐시를 전부 삭제한 뒤 다시 커밋하면 된다.

```git
git rm -r --cached .
git add .
git commit -m "fixed untracked files"
```

## 로그인 연동

이제 사용자 정보를 담당할 도메인인 User 클래스를 생성한다.

{% tabs %}
{% tab title="User.java" %}
```java
@Getter
@NoArgsConstructor
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role) {
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture) {
        this.name = name;
        this.picture = picture;

        return this;
    }

    public String getRoleKey() {
        return this.role.getKey();
    }
}
```
{% endtab %}
{% tab title="Role.java" %}
```java
@Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST","손님"),
    USER("ROLE_USER","일반 사용자");

    private final String key;
    private final String title;
}
```
{% endtab %}
{% tab title="UserRepository.java" %}
```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```
{% endtab %}
{% endtabs %}

### @Enumerated(EnumType.STRING)

JPA로 DB에 저장할 때 Enum을 어떻게 저장할지 결정한다. 기본적으로는 int 타입의 숫자가 들어가는데, 이럴 경우 DB에서 그 값이 무엇을 의미하는지 알 수가 없다. 그래서 문자열로 저장될 수 있도록 설정한다.

### ROLE
 
스프링 시큐리티에서는 권한 코드에 항상 `ROLE_`이 붙어야 한다. 

## 스프링 시큐리티 설정

{% tabs %}
{% tab title="build.gradle" %}
```groovy
compile('org.springframework.boot:spring-boot-starter-oauth2-client')
```
{% endtab %}
{% endtabs %}

소셜 로그인 등 클라이언트 입장에서 소셜 기능을 구현할 때 필요한 의존성이다. `spring-security-oauth2-client`와 `spring-security-oauth2-jose`를 기본으로 관리한다.

{% tabs %}
{% tab title="SecurityConfig.java" %}
```java
@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .headers().frameOptions().disable()
            .and()
            .authorizeRequests()
            .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**", "/profile").permitAll()
            .antMatchers("/api/v1/**").hasRole(Role.USER.name())
            .anyRequest().authenticated()
            .and()
            .logout()
            .logoutSuccessUrl("/")
            .and()
            .oauth2Login()
            .userInfoEndpoint()
            .userService(customOAuth2UserService);
    }
}
```
{% endtab %}
{% endtabs %}

### @EnableWebSecurity

Spring Security 설정을 활성화한다.

### .csrf().disable().headers().frameOptions().disable()

h2-console 화면을 사용하기 위해 해당 옵션을 disable 한다.

### authorizeRequests()

URL별로 권한을 관리할 수 있도록 설정하는 옵션의 시작점이다. `authorizeRequests()`을 선언해야만 `antMatchers`를 사용할 수 있다.

### antMatchers()

권한 관리 대상을 지정한다. URL, HTTP 메서드별로 관리할 수 있다.

`/`등 지정한 URL은 `permitAll()`로 전체 열람 권한을 주었고 `/api/v1/**`의 주소를 가진 API는 `USER` 권한을 가진 사람만 접근 가능하도록 설정했다.

### anyRequest()

설정된 값들 외의 나머지 URL을 의미한다.

위 코드에서는 `authenticated()`를 추가해 나머지 URL은 전부 인증된 사용자 즉, 로그인한 사용자만 허용하게 한다.

### logout().logoutSuccessUrl("/")

로그아웃 기능 설정을 위한 진입점이다.

여기서는 로그아웃을 성공하면 `/` 주소로 이동한다.

### oauth2Login

OAuth2 로그인 기능 설정을 위한 진입점이다.

### userInfoEndpoint

OAuth2 로그인 성공 후 사용자 정보를 가져올 때의 설정을 담당한다.

### userService

소셜 로그인에 성공하면 후속 조치를 진행할 `UserService` 인터페이스의 구현체를 등록하는 코드다.

리소스 서버인 소셜 서비스에서 사용자 정보를 가져온 상태에서 추가로 진행하려는 기능을 명시할 수 있다.

{% tabs %}
{% tab title="CustomOAuth2UserService.java" %}
```java
@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
    }

    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
}
```
{% endtab %}
{% endtabs %}

`CustomOAuth2UserService`는 구글 로그인 후에 가져온 사용자 정보(email, name, picture 등)를 기반으로 가입, 정보 수정, 세션 저장 등을 지원한다.

### registrationId

현재 로그인 진행 중인 서비스를 구분하기 위한 코드다. 지금은 구글만 사용하고 있지만 나중에 네이버 등을 추가로 연동했을 때 구글 로그인인지, 네이버 로그인인지 구분하기 위해 사용한다.

### userNameAttributeName

OAuth2 로그인을 할 때 Primary Key처럼 키가 되는 필드값을 의미한다. 구글은 `sub`이라는 기본 코드를 지원하고 네이버, 카카오 등은 지원하지 않는다. 나중에 네이버와 구글 로그인을 동시에 지원할 떄 사용할 것이다.

### OAuthAttributes

`OAuth2UserService`로 가져온 `OAuth2User`의 attribute를 담을 클래스다. 네이버 등 다른 소셜 로그인도 이 클래스를 사용할 예정이다.

### SessionUser

세션에 사용자 정보를 저장하기 위한 DTO다. `User` 클래스를 바로 쓰지 않고 `SessionUser`를 새로 만들어 사용한다.

{% hint style="info" %}
`User` 클래스를 그대로 사용하면 안 되는 이유
{% endhint %}

`User` 클래스를 그대로 사용하면 아래와 같은 에러가 발생한다.

```text
Failed to convert from type [java.lang.Object] to type [byte[]] for value 'domain.user.User@4a43d6'
```

`User` 클래스를 세션에 저장하려고 하자 `User` 클래스에 직렬화를 구현하지 않았다는 의미다. 그럼 `User` 클래스에 직렬화 코드를 넣으면 해결될까? 아니다. `User` 클래스는 엔티티이기 때문이다. 

엔티티 클래스는 언제 다른 엔티티와 관계가 형성될지 모른다. 만약 `@OneToMany` 등으로 인해 자식 엔티티를 갖고 있다면 자식 엔티티도 직렬화를 시켜주어야 하므로 성능 이슈나 생각하지 못한 상황이 발생할 수 있다. 따라서 직렬화 기능을 가진 DTO를 따로 만들어주는 게 운영, 유지보수를 위해 좋다.

### saveOrUpdate()

구글 사용자 정보가 업데이트 될 때를 대비해 update를 같이 구현했다. 사용자 이름이나 프로필 사진이 변경되면 `User` 엔티티에도 반영된다.

{% tabs %}
{% tab title="OAuthAttributes.java" %}
```java
@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity() {
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}
```
{% endtab %}
{% endtabs %}

### of()

`OAuth2User`에서 반환하는 사용자 정보는 Map이기 때문에 값을 하나하나 변환해야 한다.

### toEntity()

처음 가입할 시점에 `OAuthAttributes`에서 `User` 엔티티를 생성한다. 기본 권한은 `GUEST`로 설정한다. `OAuthAttributes` 클래스 생성이 끝나면 같은 패키지의 `SessionUser` 클래스를 생성한다.

{% tabs %}
{% tab title="SessionUser.java" %}
```java
@Getter
public class SessionUser implements Serializable {
    private String name;
    private String email;
    private String picture;
    
    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}
```
{% endtab %}
{% endtabs %}

`SessionUser`에는 인증된 사용자 정보만 필요하다.

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

`CustomOAuth2UserService`는 로그인에 성공하면 세션에 `SessionUser`를 저장한다. 즉, 로그인 성공 시 `httpSession.getAttribute()`로 값을 가져올 수 있는 것이다.