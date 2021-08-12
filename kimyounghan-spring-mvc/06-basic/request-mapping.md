# 요청 매핑

## @RequestMapping

```java

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/hello-basic")
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }
}
```

- 배열을 이용해 다중 설정을 제공할 수 있다.
    - `@RequestMapping({"/hello-basic", "/hello-go"})`
- 허용 범위
    - `/hello-basic`으로 매핑해놓고 `/hello-basic/`으로 요청해도 같은 요청으로 매핑한다.
- 메서드 속성
    - GET 등 HTTP 메서드를 지정하지 않으면 뭘로 요청하든 무관하게 호출된다.

## HTTP 메서드 매핑

```java

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
    public String mappingGetV1() {
        log.info("mappingGetV1");
        return "ok";
    }

    @GetMapping(value = "/mapping-get-v2")
    public String mappingGetV2() {
        log.info("mapping-get-v2");
        return "ok";
    }
}
```

- 해당 HTTP 메서드로 요청하지 않으면 `405 Method Not Allowed`를 보낸다.

## 경로 변수(PathVariable)

```java

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    /*
     * /mapping/userA 이런 식으로 경로 자체에 값이 있는 것
     * */
    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data) {
        log.info("mappingPath userId={}", data);
        return "ok";
    }

    /* 변수명이 같으면 생략 가능하다. */
    @GetMapping("/mapping/{userId}")
    public String mappingPath2(@PathVariable String userId) {
        log.info("mappingPath userId={}", userId);
        return "ok";
    }
}

```

- 최근 HTTP API는 이처럼 리소스 경로에 식별자를 넣는 스타일을 선호한다.
    - `/users/1`
    - `?`를 넣는 쿼리 스트링과는 다르다.
- @PathVariable의 이름과 파라미터의 이름이 같으면 생략할 수 있다.
    - 그렇다고 애너테이션 자체를 생략할 수는 없다.

### 다중 사용

```java

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    @GetMapping("/mapping/users/{userId}/orders/{orderId}")
    public String mappingPath(@PathVariable String userId, @PathVariable Long orderId) {
        log.info("mappingPath userId={}, orderId={}", userId, orderId);
        return "ok";
    }
}

```

- 다중 사용도 가능하다.

### 파라미터 조건 매핑

```java

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    /**
     * 파라미터로 추가 매핑
     * params="mode",
     * params="!mode"
     * params="mode=debug"
     * params="mode!=debug" (! = )
     * params = {"mode=debug","data=good"}
     */
    @GetMapping(value = "/mapping-param", params = "mode=debug")
    public String mappingParam() {
        log.info("mappingParam");
        return "ok";
    }
}

```

- 파라미터에 조건을 걸 수도 있다.
- 예시는 `http://localhost:8080/mapping-param?mode=debug` 처럼 파라미터에 설정한 문자열이 있어야 매핑된다.
- `mode=debug`라는 문자열 없이 요청하면 `400 Bad Request`가 뜬다.
- 경로 변수 방식을 많이 사용하는 추세라 잘 쓰지 않는다.

### 헤더 조건 매핑

```java

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    /**
     * 특정 헤더로 추가 매핑
     * headers="mode",
     * headers="!mode"
     * headers="mode=debug"
     * headers="mode!=debug" (! = )
     */
    @GetMapping(value = "/mapping-header", headers = "mode=debug")
    public String mappingHeader() {
        log.info("mappingHeader");
        return "ok";
    }
}

```

- 헤더에 특정 문자가 있어야만 매핑된다.
- 역시 잘 쓰지는 않는다.

### 미디어 타입 조건 매핑

```java

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    /**
     * Content-Type 헤더 기반 추가 매핑 Media Type * consumes="application/json"
     * consumes="!application/json"
     * consumes="application/*"
     * consumes="*\/*"
     * MediaType.APPLICATION_JSON_VALUE
     */
    @PostMapping(value = "/mapping-consume", consumes = "application/json")
    public String mappingConsumes() {
        log.info("mappingConsumes");
        return "ok";
    }
}

```

- header의 content-type에 따라 매핑한다.
- Content-type always is about the content of the current request or response.
- html인지 등 타입에 따라 쓰려면 headers가 아니라 consumes를 사용해야 한다.
    - 내부에서 처리하는 내용이 다르다.

```java

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    /**
     * Accept 헤더 기반 Media Type
     * produces = "text/html"
     * produces = "!text/html"
     * produces = "text/*"
     * produces = "*\/*"
     */
    @PostMapping(value = "/mapping-produce", produces = "text/html")
    public String mappingProduces() {
        log.info("mappingProduces");
        return "ok";
    }
}

```

- Accept 헤더의 미디어 타입을 매핑한다.
- 웹 브라우저 측에서 받아들일 수 있는 타입을 이야기 한다.
    - Accept indicates what kind of response from the server the client can accept.
    - 예시는 `클라이언트는 text/html 이란 타입을 받아들일 수 있다` 는 뜻이다.

## 예시

- 회원 목록 조회
    - GET
    - /users
- 회원 등록
    - POST
    - /users
- 회원 조회
    - GET
    - /users/{userId}
- 회원 수정
    - PATCH
    - /users/{userId}
- 회원 삭제
    - DELETE
    - /users/{userId}

```java
@RestController
@RequestMapping("/mapping/users")
public class MappingClassController {

    @GetMapping
    public String user() {
        return "get users";
    }

    @PostMapping
    public String addUser() {
        return "post user";
    }

    @GetMapping("/{userId}")
    public String findUser(@PathVariable String userId) {
        return "get userId = " + userId;
    }

    @PatchMapping("/{userId}")
    public String updateUser(@PathVariable String userId) {
        return "update userId = " + userId;
    }

    @DeleteMapping("/{userId}")
    public String deleteUser(@PathVariable String userId) {
        return "delete userId = " + userId;
    }
}
```