# HTTP 요청 파라미터

클라이언트에서 서버로 데이터를 보낼 때는 주로 3가지 방법을 사용한다.

- GET
    - 쿼리 파라미터
        - /url?username=hello&age=20
    - 메시지 바디 없이 URL 쿼리 파라미터에 데이터를 포함해 전달한다.
    - 검색, 필터, 페이징 등에서 사용한다.
- POST
    - HTML Form
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파라미터 형식으로 전달한다.
        - username=hello&age=20
    - 회원 가입, 상품 주문, HTML Form에 사용한다.
- HTTP message body에 직접 담아서 요청
    - HTTP API에 주로 사용한다.
        - json, xml, text
        - 주로 json 데이터를 사용한다.
    - POST, PUT, PATCH에 쓰인다.

## 쿼리 파라미터, HTML Form

```text
# GET 쿼리 파라미터
http://localhost:8080/request-param?username=hello&age=20

# POST HTML Form 전송
POST /request-param ...
content-type: application/x-www-form-urlencoded
username=hello&age=20
```

- 요청 파랄미터 조회라고도 한다.
- HttpServletRequest의 request.getParameter()
    - GET 쿼리 파라미터든 POST HTML Form이든 둘 다 형식이 같으므로 구분없이 조회할 수 있다.

```java

@Slf4j
@Controller
public class RequestParamController {

    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        log.info("username = {}, age = {}", username, age);

        response.getWriter().write("ok");
    }
}
```

- request.getParameter()로 요청 파라미터를 조회한다.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/request-param-v1" method="post">
    username: <input type="text" name="username"/> age: <input type="text" name="age"/>
    <button type="submit">전송</button>
</form>
</body>
</html>
```

```text
2022-03-01 00:10:55.961  INFO 23155 --- [nio-8080-exec-2] h.s.b.request.RequestParamController     : username = 도던, age = 33
```

- HTML Form으로 요청 파라미터를 조회한다.

## @RequestParam

```java

@Slf4j
@Controller
public class RequestParamController {

    // RestController처럼 스트링 값을 뷰가 아니라 문자 그래도 반환하도록 해준다.
    @ResponseBody
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge
    ) {
        log.info("memberName = {}, memberAge = {}", memberName, memberAge);

        return "ok";
    }
}
```

```text
2022-03-01 13:23:16.760  INFO 76287 --- [nio-8080-exec-1] h.s.b.request.RequestParamController     : memberName = hello, memberAge = 20
```

- @RequestParam
    - 파라미터 이름으로 바인딩 한다.
    - GET 쿼리 파라미터든, HTML Form이든 파라미터를 잘 받아온다.
- @ResponseBody
    - view 조회 대신 HTTP message body에 직접 입력한다.

```text
@RequestParam("username") String memberName = request.getParameter("username")
```

- @RequestParam의 name(value) 속성이 파라미터와 일치해야 한다.

```java

@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/request-param-v3")
    public String requestParamV3(
            @RequestParam String username,
            @RequestParam int age
    ) {
        log.info("username = {}, age = {}", username, age);

        return "ok";
    }
}

```

- 이름과 변수명이 같다면 생략할 수 있다.

````java

@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/request-param-v4")
    public String requestParamV4(String username, int age) {
        log.info("username = {}, age = {}", username, age);

        return "ok";
    }
}
````

- 이름과 변수명이 같고 String, int, Integer 등의 단순한 타입이면 @RequestParam 애너테이션까지 생략할 수 있다.
- 너무 다 생략하면 과할 수 있으니 애너테이션 정도는 붙여줘서 요청 파라미터를 읽는다고 명확하게 표시해주자.

### 필수값 지정

```java

@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/request-param-required")
    public String requestParamRequired(
            @RequestParam(required = true) String username,
            @RequestParam(required = false) int age
    ) {
        log.info("username = {}, age = {}", username, age);

        return "ok";
    }
}
```

```text
# 쿼리 스트링에서 username을 뺐을 때
2022-03-01 13:36:44.587WARN 77398---[nio-8080-exec-4].w.s.m.s.DefaultHandlerExceptionResolver:Resolved
[org.springframework.web.bind.MissingServletRequestParameterException:Required String parameter'username'is not present]

# 쿼리 스트링에서 age를 뺐을 때
java.lang.IllegalStateException:Optional int parameter'age'is present but cannot be translated into a null value due to being declared as a primitive type.
Consider declaring it as object wrapper for the corresponding primitive type.
```

- required 옵션은 기본적으로 true다.
- 필수값이 아니면 false로 지정한다.
    - 필수값을 넣지 않으면 400 에러가 발생한다.
    - null이 아니라 `?username=`처럼 빈 값이 들어오는 경우라면 ok가 떨어지므로 주의한다.
- 필수가 아니더라도 primitive type이면 null이 들어갈 수 없으므로 Wrapper 클래스로 바꿔줘야 한다.

```java

@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/request-param-default")
    public String requestParamDefault(
            @RequestParam(required = true, defaultValue = "guest") String username,
            @RequestParam(required = false, defaultValue = "-1") int age
    ) {
        log.info("username = {}, age = {}", username, age);

        return "ok";
    }
}
```

- required 상관없이 정상 동작하게 하려면 defaultValue를 사용할 수 있다.
- 빈 문자도 기본값을 적용해준다.

```java

@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/request-param-map")
    public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
        log.info("username = {}, age = {}", paramMap.get("username"), paramMap.get("age"));

        return "ok";
    }
}
```

- 파라미터를 한 번에 전부 받고 싶다면 Map을 사용할 수 있다.
- @RequestParam Map
    - 값이 확실하게 1개인 경우에 사용한다.
    - Map(key=value)
- @RequestParam MultiValueMap
    - MultiValueMap(key=[value1, value2, ...])
    - ex. (key=userIds, value=[id1, id2])

## @ModelAttribute

```java

@Controller
class ExampleController {

    @RequestMapping
    void example(
            @RequestParam String username,
            @RequestParam int age
    ) {
        HelloData data = new HelloData();

        data.setUsername(username);
        data.setAge(age);
    }
}
```

- 실제 개발 하면 요청 파라미터를 다시 객체에 넣는 과정을 반복한다.
- @ModelAttribute를 사용하면 이 과정을 자동화할 수 있다.

```java

@Data
public class HelloData {
    private String username;
    private int age;
}

@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/model-attribute-v1")
    public String modelAttributeV1(@ModelAttribute HelloData helloData) {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }
}
```

1. HelloData라는 객체를 생성한다.
2. 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다.
3. 해당 프로퍼티의 setter를 호출해 파라미터의 값을 바인딩한다.
    - 파라미터가 username이면 setUsername()을 호출해 값을 입력한다.
    - 문자에 숫자가 들어오거나 한다면 BindException이 발생한다.

### @ModelAttribute 생략

```java

@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/model-attribute-v2")
    public String modelAttributeV2(HelloData helloData) {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }
}
```

- 애너테이션을 빼도 작동한다.
- 객체 이름은 상관 없이 그 내부의 프로퍼티 이름만 맞으면 된다.
- @RequestParam 또한 생략 가능해 혼란스러우므로 붙여주는 게 좋다.

애너테이션 생략 시 규칙은 다음과 같다.

- @RequestParam
    - String, int, Integer 등 단순한 타입
- @ModelAttribute
    - 나머지
    - argument resolver로 지정해둔 타입 외의 것들
        - ex. HttpServletResponse
        