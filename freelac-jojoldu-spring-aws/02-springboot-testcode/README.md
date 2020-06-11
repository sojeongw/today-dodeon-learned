# 스프링부트에서 테스트 코드를 작성하자

## 테스트 코드의 이점

- 개발 단계 초기에 문제를 발견할 수 있다.
- 나중에 리팩토링 하거나 라이브러리를 업그레이드할 때 올바르게 동작하는지 확인할 수 있다.
- 기능에 대한 불확실성을 감소시킬 수 있다.
- 단위 테스트 자체가 문서로 상요될 수 있다.

## 테스트 코드 작성하기

{% tabs %}
{% tab title="Application.java" %}
```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        // 내장 WAS 실행
        SpringApplication.run(Application.class, args);
    }

}
```
{% endtab %}
{% endtabs %}

이 `Application` 클래스는 앞으로 만들 프로젝트의 메인 클래스다. 

### @SpringBootApplication

스프링 부트 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 해준다. `@SpringBootApplication`이 있는 위치부터 설정을 읽어가므로 항상 프로젝트 최상단에 있어야 한다.

### 내장 WAS

외부에 별도 WAS를 두지 않고 내부에서 실행한다. 이렇게 하면 항상 서버에 톰캣을 설치할 필요가 없으며, 스프링 부트로 만들어진 jar파일(실행 가능한 java 패키징 파일)로 실행할 수 있다.

스프링 부트는 내장 WAS 사용을 권장한다. 언제 어디서나 같은 환경에서 배포할 수 있기 때문이다. 외장 WAS를 사용하면 모든 서버가 WAS의 종류, 버전, 설청을 동일하게 가져가야 하므로 큰 작업이 될 수 있다. 

{% tabs %}
{% tab title="HelloController.java" %}
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```
{% endtab %}
{% endtabs %}

### @RestController

JSON을 반환하는 컨트롤러로 만들어준다. 예전에 `@ResponseBody`를 각 메서드마다 선언했던 것을 한 번에 사용할 수 있게 해준다.

### @GetMapping

HTTP 메서드인 Get 요청을 받는 API로 설정한다. 과거에는 `@RequestMapping(method = RequestMethod.GET)`으로 표현했던 부분이다.

{% tabs %}
{% tab title="HelloControllerTest.java" %}
```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}
```
{% endtab %}
{% endtabs %}

### @RunWith(SpringRunner.class)

테스트 실행 시 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다. 스프링부트 테스트와 JUnit 사이에 연결자 역할을 한다고 볼 수 있다. 이 코드에서는 `SpringRunner`라는 스프링 실행자를 사용한다.

### @WebMvcTest(controllers = HelloController.class)

다양한 스프링 애너테이션 중 Web(Spring MVC)에 집중할 수 있는 애너테이션이다. 

선언할 경우 `@Controller`, `@ControllerAdvice` 등을 사용할 수 있다. 하지만 `@Service`, `@Component`, `@Repository`는 쓸 수 없다. 여기서는 컨트롤러만 사용하기 때문에 `@WebMvcTest`를 사용했다.

### @Autowired

스프링이 관리하는 빈을 주입받는다.

### MockMvc

웹 API를 테스트할 때 사용하는 클래스로 HTTP GET, POST 등에 대한 API 테스트를 할 수 있다. 스프링 MVC 테스트의 시작점이다.

### mvc.perform(get("/hello"))

`MockMvc`를 통해 /hello 주소로 HTTP GET 요청을 한다.

- `.andExpect(status().isOk())`
    - `mvc.perform`의 결과, HTTP 헤더의 status를 검증한다.
    - 200, 404, 500 등의 상태를 검증하는데 여기선 200인지 확인한다.
- `.andExpect(content().string(hello))`
    - 응답 본문의 내용을 검증한다.
    - 이 코드는 컨트롤러에서 'hello'를 리턴하는지 검증한다.
    
## 롬복으로 전환하기

{% tabs %}
{% tab title="HelloResponseDto.java" %}
```java
@RequiredArgsConstructor
@Getter
public class HelloResponseDto {
    private final String name;
    private final int amount;
}
```
{% endtab %}
{% endtabs %}

### @RequiredArgsConstructor

선언된 모든 final 필드가 포함된 생성자를 생성한다. final이 없는 필드는 생성자에 포함되지 않는다.

{% tabs %}
{% tab title="HelloResponseDtoTest.java" %}
```java
public class HelloResponseDtoTest {
    @Test
    public void 롬복_기능_테스트() {
        // given
        String name = "test";
        int amount = 1000;

        // when
        HelloResponseDto helloResponseDto = new HelloResponseDto(name, amount);

        // then
        assertThat(helloResponseDto.getName()).isEqualTo(name);
        assertThat(helloResponseDto.getAmount()).isEqualTo(amount);
    }
}
```
{% endtab %}
{% endtabs %}

### assertThat

`assertj`라는 테스트 검증 라이브러리의 검증 메서드로, 검증하고 싶은 대상을 인자로 받는다. 메서드 체이닝이 지원되어서 `isEqualTo`와 같은 메서드를 이어서 사용할 수 있다.

`JUnit`에도 `assertThat`이 있음에도 불구하고 `assertj`를 사용한 이유는 다음과 같다.

- 추가적인 라이브러리가 필요없다.
    - `JUnit`의 `assertThat`을 쓰면 `is()`와 같은 `CoreMatchers` 라이브러리가 필요하다.
- 자동 완성이 더 잘 지원된다.
    - IDE는 `CoreMatchers`같은 `Matcher` 라이브러리의 자동 완성 지원이 빈약하다.

{% tabs %}
{% tab title="HelloController.java" %}
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    // ResponseDto를 적용한 코드
    @GetMapping("/hello/dto")
    // 외부에서 name이란 이름으로 넘긴 파라미터를 메서드 파라미터인 String name에 저장한다.
    public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
}
```
{% endtab %}
{% endtabs %}

### @RequestParam

외부에서 API로 넘긴 파라미터를 가져온다.

{% tabs %}
{% tab title="HelloControllerTest.java" %}
```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk()) 
                .andExpect(content().string(hello));
    }

    @Test
    public void helloDto가_리턴된다() throws Exception {
        String name = "hello";
        int amount = 1000;

        mvc.perform(get("/hello/dto")
                .param("name", name)
                .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is(name)))
                .andExpect(jsonPath("$.amount", is(amount)));


    }
}
```
{% endtab %}
{% endtabs %}

### .param("name", name)

API 테스트 시 사용할 요청 파라미터를 설정한다. String만 허용하기 때문에 숫자, 날짜 등의 데이터도 등록할 때는 문자열로 변환해야 한다.

### jsonPath("$.name", is(name))

JSON 응답값을 필드별로 검증한다. `$`를 기준으로 필드명을 적는데, 여기서는 `name`과 `amount`를 검증하고 있다.