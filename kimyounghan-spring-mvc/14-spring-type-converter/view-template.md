# 뷰 템플릿에 적용하기

{% tabs %} {% tab title="ConverterController.java" %}

```java

@Controller
public class ConverterController {

    @GetMapping("/converter-view")
    public String converterView(Model model) {
        model.addAttribute("number", 10000);
        model.addAttribute("ipPort", new IpPort("127.0.0.1", 8080));

        return "converter-view";
    }
}
```

{% endtab %} {% tab title="converter-view.html" %}

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Title</title>
</head>
<body>
<ul>
<li>${number}:<span th:text="${number}"></span></li>
<li>${{number}}:<span th:text="${{number}}"></span></li>
<li>${ipPort}:<span th:text="${ipPort}"></span></li>
<li>${{ipPort}}:<span th:text="${{ipPort}}"></span></li>
</ul>
</body>
</html>
```

{% endtab %} {% endtabs %}

```text
${number}: 10000
${{number}}: 10000
${ipPort}: hello.typeconverter.type.IpPort@59cb0946
${{ipPort}}: 127.0.0.1:8080
```

![](../../.gitbook/assets/kimyounghan-spring-mvc/14/screenshot%202022-03-26%20오후%208.24.57.png)

뷰 템플릿에도 `{{number}}` 형태로 데이터를 문자로 컨버팅 해 출력할 수 있다.

## Form에 적용하기

{% tabs %} {% tab title=".java" %}

```java

@Controller
public class ConverterController {

    @GetMapping("/converter/edit")
    public String converterForm(Model model) {
        IpPort ipPort = new IpPort("127.0.0.1", 8080);
        Form form = new Form(ipPort);
        model.addAttribute("form", form);
        return "converter-form";
    }

    @PostMapping("/converter/edit")
    public String converterEdit(@ModelAttribute Form form, Model model) {
        IpPort ipPort = form.getIpPort();
        model.addAttribute("ipPort", ipPort);
        return "converter-view";
    }

    @Data
    static class Form {
        private IpPort ipPort;

        public Form(IpPort ipPort) {
            this.ipPort = ipPort;
        }
    }
}
```

{% endtab %} {% tab title="converter-form.html" %}

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Title</title>
</head>
<body>
<form th:object="${form}"th:method="post">
        th:field<input type="text"th:field="*{ipPort}"><br/>
        th:value<input type="text"th:value="*{ipPort}">(보여주기 용도)<br/>
<input type="submit"/>
</form>
</body>
</html>
```

{% endtab %} {% endtabs %}

Form에도 적용 가능하다.

- converterForm()
    - th:field가 자동으로 컨버전 서비스를 적용해 `${{ipPort}}`를 해준 것처럼 IpPort에서 String 타입이 되었다.
- converterEdit()
    - @ModelAttribute가 내부적으로 컨버전 서비스를 사용해 String에서 IpPort로 변환한다.