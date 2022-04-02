# 기존 테스트에 시큐리티 적용하기

시큐리티 옵션이 활성화되면 인증된 사용자만 API를 호출할 수 있다. 기존 API 테스트 코드들은 모두 인증 권한을 받지 못했으므로 수정이 필요하다.

## HelloControllerTest - CustomOAuth2UserService를 찾을 수 없음

![](../../.gitbook/assets/freelac-jojoldu-spring-aws/05/스크린샷%202020-09-13%20오후%206.50.49.png)

`CustomOAuth2UserService`를 생성하는데 필요한 소셜 로그인 설정값이 없기 때문에 발생한다.

```properties
spring.security.oauth2.client.registration.google.client-id=클라이언트 ID
spring.security.oauth2.client.registration.google.client-secret=클라이언트 보안 비밀번호
spring.security.oauth2.client.registration.google.scope=profile, email
```

`application-oauth-properties`에 위처럼 추가했음에도 오류가 나는 이유는 환경의 차이 때문이다. `src/main`과 `src/test`는 각자만의 환경 구성을 가진다. 

다만, `src/main/resources/application.properties`가 테스트에도 적용되는 이유는 test에 `application.properties`가 없으면 main의 설정을 가져오기 때문이다. 자동으로 가져오는 옵션의 범위는 오직 이 `application.properties`뿐이다.

따라서 우리는 테스트를 위한 가짜 설정 파일을 만들어야 한다.

{% tabs %}
{% tab title="test/resources/application.properties" %}
```properties
spring.jpa.show_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.h2.console.enabled=true
spring.session.store-type=jdbc

# Test OAuth

spring.security.oauth2.client.registration.google.client-id=test
spring.security.oauth2.client.registration.google.client-secret=test
spring.security.oauth2.client.registration.google.scope=profile,email
```
{% endtab %}
{% endtabs %}

## PostsApiControllerTest - 302 FOUND

![](../../.gitbook/assets/freelac-jojoldu-spring-aws/05/스크린샷%202020-09-13%20오후%207.00.09.png)

스프링 시큐리티 설정 때문에 인증되지 않은 사용자의 요청은 리다이렉트되었다. 임의로 인증된 사용자를 추가해 API만 테스트할 수 있게 해보자.

{% tabs %}
{% tab title="build.gradle" %}
```groovy
dependencies {
    compile('org.springframework.security:spring-security-test')
}
```
{% endtab %}
{% tab title="PostsApiControllerTest.java" %}
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    ...

    @Autowired
    private WebApplicationContext context;

    private MockMvc mvc;

    @After
    public void tearDown() throws Exception {
        postsRepository.deleteAll();
    }

    // mockMvc 사용
    @Before
    public void setup() {
        mvc = MockMvcBuilders
                .webAppContextSetup(context)
                .apply(springSecurity())
                .build();
    }

    @Test
    @WithMockUser(roles = "USER")   // 애너테이션 추가
    public void Posts_등록된다() throws Exception {
        // given
        ...

        // when
        mvc.perform(post(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());

        // then
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }

    @Test
    @WithMockUser(roles = "USER")
    public void Posts_수정된다() throws Exception {
        // given
       ...

        // when
        mvc.perform(put(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());

        // then
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }
}
```
{% endtab %}
{% endtabs %}

`spring-security-test` 의존성을 추가하고 `@WithMockUser` 애너테이션을 추가한다. `@WithMockUser`는 인증된 가짜 사용자를 만들어 roles에 권한을 부여할 수 있다. 즉, `ROLE_USER` 권한을 가진 사용자가 API를 요청하는 것과 동일한 효과를 가진다.


또한, `@WithMockUser`는 `MockMvc`에서만 작동하므로 코드를 위와 같이 변경한다. `mvc.perform()`은 생성된 `MockMvc`로 API를 테스트한다. 본문인 Body 영역은 문자열로 표현하기 위해 `ObjectMapper`를 이용해 문자열 JSON으로 변환한다.

## HelloControllerTest - CustomOAuth2UserService를 찾을 수 없음

![](../../.gitbook/assets/freelac-jojoldu-spring-aws/05/스크린샷%202020-09-13%20오후%206.50.49.png)

맨 처음에 `CustomOAuth2UserService` 이슈로 설정값을 추가했음에도 여전히 같은 오류가 뜬다. 스프링 시큐리티 설정은 잘 작동했지만 `@WebMvcTest`는 `CustomOAuth2UserService`를 스캔하지 않기 때문이다.

`@WebMvcTest`는 `WebSecurityConfigurerAdapter`, `WebMvcConfigurer`, `@ControllerAdvice`, `@Controller`를 읽는다. 즉, `@Respository`, `@Service`, `@Component`는 스캔 대상이 아니다. `SecurityConfig`는 읽었지만 `SecurityConfig` 생성에 필요한 `CustomOAuth2UserService`를 읽지 못해 같은 에러가 발생한 것이다. 

{% tabs %}
{% tab title="HelloControllerTest.java" %}
```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class,
excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
})
public class HelloControllerTest {
    ...

    @Test
    @WithMockUser(roles = "USER")
    public void hello가_리턴된다() throws Exception {
        ...
    }

    @Test
    @WithMockUser(roles = "USER")
    public void helloDto가_리턴된다() throws Exception {
        ...
    }
}
```
{% endtab %}
{% endtabs %}

따라서 `SecurityConfig`를 스캔 대상에서 제거하고 `@WithMockUser`를 사용해 인증된 사용자를 생성한다.

![](../../.gitbook/assets/freelac-jojoldu-spring-aws/05/스크린샷%202020-09-13%20오후%207.30.12.png)

다시 실행하면 이런 에러가 발생한다. `@EnableJpaAuditing`을 사용하기 위해서는 최소 하나의 `@Entity` 클래스가 필요하다. 이 테스트는 `@WebMvcTest`이므로 당연히 Entity가 존재하지 않는다.

`@EnableJpaAuditing`가 `@SpringBootApplication`과 함께 있다보니, `@WebMvcTest`에서도 스캔할 수밖에 없다. `@EnableJpaAuditing`을 `@SpringBootApplication`에서 분리해보자.

{% tabs %}
{% tab title="Application.java" %}
```java
// @EnableJpaAuditing 삭제
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
{% endtab %}
{% tab title="JpaConfig.java" %}
```java
@Configuration
@EnableJpaAuditing  // JPA Auditing 활성화
public class JpaConfig {
}
```
{% endtab %}
{% endtabs %}

`Application.java`에서 삭제한 뒤 `JpaConfig`를 새로 만들어 추가해주면 모든 테스트가 잘 동작한다.