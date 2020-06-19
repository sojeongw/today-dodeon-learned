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
</body>
</html>
```
{% endtab %}
{% endtabs %}

페이지 로딩 속도를 높이기 위해 css는 header에, js는 footer에 두었다. HTML은 위에서부터 코드가 실행되기 때문에 head가 다 실행되고 나서 body가 실행된다. 즉, head가 불러지지 않으면 백지만 노출된다. js의 용량이 클수록 body의 실행이 늦어지기 때문에 js는 body 하단에 두고 화면이 다 그려진 뒤에 호출하는 게 좋다.

반면 css는 화면을 그리는 역할이므로 head에서 불러오는 게 좋다. 그렇지 않으면 사용자는 css가 적용되지 않은 화면을 볼 수 있기 때문이다.

bootstrap.js는 제이쿼리가 꼭 있어야 하기 때문에 부트스트랩보다 먼저 호출되도록 작성했다. 이 상황을 bootstrap.js가 제이쿼리에 의존한다고 표현한다.

이제 라이브러리와 기타 HTML 태그가 모두 레이아웃에 추가되어 `index.mustache`에는 필요한 코드만 남게 되었다. 파일을 분리했으니 글 등록 버튼을 하나 추가한다.

{% tabs %}
{% tab title="index.mustache" %}
```html
{{>layout/header}}
<h1>스프링 부트 웹서비스</h1>
<div class="col-md-12">
    <div class="row">
        <div class="col-md-6">
            <a href="posts/save" role="button" class="btn btn-primary">글 등록</a>
        </div>
    </div>
</div>
{{>layout/footer}}
```
{% endtab %}
{% endtabs %}

`{{>}}`는 현재 머스테치 파일(여기서는 `index.mustache`)을 기준으로 다른 파일을 가져온다.

`<a>` 태그를 통해 글 등록 페이지로 이동한다. 이동할 페이지의 주소는 `/posts/save`가 된다.

{% tabs %}
{% tab title="IndexController.java" %}
```java
@Controller
@RequiredArgsConstructor
public class IndexController {

    ...

    @GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }
}
```
{% endtab %}
{% tab title="posts-save.mustache" %}
```html
{{>layout/header}}

<h1>게시글 등록</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
            </div>
            <div class="form-group">
                <label for="author"> 작성자 </label>
                <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
            </div>
            <div class="form-group">
                <label for="content"> 내용 </label>
                <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-save">등록</button>
    </div>
</div>

{{>layout/footer}}
```
{% endtab %}
{% tab title="index.js" %}
```javascript 1.5
var main = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });

        $('#btn-update').on('click', function () {
            _this.update();
        });

        $('#btn-delete').on('click', function () {
            _this.delete();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    update : function () {
        var data = {
            title: $('#title').val(),
            content: $('#content').val()
        };

        var id = $('#id').val();

        $.ajax({
            type: 'PUT',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    delete : function () {
        var id = $('#id').val();

        $.ajax({
            type: 'DELETE',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8'
        }).done(function() {
            alert('글이 삭제되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }

};

main.init();
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

`/posts/save`에 해당하는 컨트롤러를 생성하고 이동할 페이지를 만든다. 이 화면에 있는 버튼에 기능을 추가하기 위해 JS파일인 `index.js`도 구현한다. `footer.mustache`엔 `index.js`를 머스테치 파일이 쓸 수 있게 추가해준다.

### 정적 파일 URL

`index.js` 호출 코드를 보면 절대 경로 `/`로 바로 시작한다. 스프링 부트에서 `src/main/resources/static`에 위치한 자바스크립트, CSS, 이미지 등의 정적 파일은 URL에서 `/`로 설정된다.

- src/main/resources/static/js/...
    - http://도메인/js/...
- src/main/resources/static/css/...
    - http://도메인/css/...
- src/main/resources/static/image/...
    - http://도메인/image/...