# 기본 페이지 만들기

{% tabs %}
{% tab title="build.gradle" %}
```groovy
dependencies {
    ...
    compile('org.springframework.boot:spring-boot-starter-mustache')
}
```
{% endtab %}
{% endtabs %}

먼저 머스테치 스타터 의존성을 등록한다.

{% tabs %}
{% tab title="index.mustache" %}
```html
<!DOCTYPE HTML>

<html>
<head>
    <title>스프링 부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8"/>
</head>
<body>
    <h1>스프링 부트로 시작하는 웹 서비스</h1>
</body>
</html>
```
{% endtab %}
{% tab title="IndexController.java" %}
```html
@Controller
public class IndexController {
    
    @GetMapping("/")
    public String index() {
        return "index";
    }
}
```
{% endtab %}
{% endtabs %}

머스테치의 파일 위치는 기본적으로 `src/main/resources/templates`다. 이 위치에 첫 페이지인 `index.mustache`를 생성한다.

이 머스테치에 URL을 매핑하기 위해 `IndexController`를 생성한다. 머스테치 스타터 덕분에 컨트롤러에서 문자열을 반환하면 앞의 경로와 뒤의 파일 확장자는 자동으로 지정된다. 즉, `src/main/resources/templates/index.mustache`로 전환되어 View Resolver가 처리한다.

{% tabs %}
{% tab title="index.mustache" %}
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class IndexControllerTest {
    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void 메인페이지_로딩() {
        // given
        String body = this.restTemplate.getForObject("/", String.class);
        
        // then
        assertThat(body).contains("스프링 부트로 시작하는 웹 서비스");
    }
}
```
{% endtab %}
{% endtabs %}

이번 테스트는 URL을 호출하면 페이지 내용이 제대로 호출되는지 확인한다. HTML도 결국 규칙이 있는 문자열이기 때문에 호출했을 때 `index.mustache`에 포함된 코드가 반환되는지 확인하면 된다.

## 게시글 등록 화면 만들기

오픈 소스인 부트스트랩을 이용해 화면을 만든다. 부트스트랩과 같은 프론트엔드 라이브러리를 사용하는 방법은 2가지가 있다. 하나는 외부 CDN을 사용하는 것이고, 다른 하나는 직접 라이브러리를 받는 것이다. 

이 프로젝트에서는 전자를 사용한다. 이 방법은 외부 서비스에 의존하게 되어 CDN 서비스에 문제가 생기면 영향을 받기 때문에 실제 서비스에서는 사용하지 않는다.

부트스트랩은 레이아웃 방식으로 추가한다. 공통 영역을 별도의 파일로 분리해 필요한 곳에서 가져다 쓰는 방식을 말한다. 부트스트랩과 제이쿼리는 머스테치 화면 어디서든 필요하기 때문에 매번 추가하는 것이 귀찮으므로 레이아웃 파일을 만들어 추가한다.

{% tabs %}
{% tab title="header.mustache" %}
```html
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body>
```
{% endtab %}
{% tab title="footer.mustache" %}
```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

<!--index.js 추가-->
<script src="/js/app/index.js"></script>
</body>
</html>
```
{% endtab %}
{% endtabs %}

페이지 로딩 속도를 높이기 위해 css는 header에, js는 footer에 두었다. HTML은 위에서부터 코드가 실행되기 때문에 head가 다 실행되고 나서 body가 실행된다. 즉, head가 불러지지 않으면 백지만 노출된다. js의 용량이 클수록 body의 실행이 늦어지기 때문에 js는 body 하단에 두고 화면이 다 그려진 뒤에 호출하는 게 좋다.

반면 css는 화면을 그리는 역할이므로 head에서 불러오는 게 좋다. 그렇지 않으면 사용자는 css가 적용되지 않은 화면을 볼 수 있기 때문이다.

bootstrap.js는 제이쿼리가 꼭 있어야 하기 때문에 부트스트랩보다 먼저 호출되도록 작성했다. 이 상황을 bootstrap.js가 제이쿼리에 의존한다고 표현한다.

(작성중)