# 스프링 타입 컨버터

{% tabs %} {% tab title="HelloController.java" %}

```java

@RestController
public class HelloController {

    @GetMapping("/hello-v2")
    public String helloV2(@RequestParam Integer data) {
        System.out.println("data = " + data);
        return "ok";
    }
}
```

{% endtab %} {% endtabs %}

```text
http://localhost:8080/hello-v2?data=10
```

- 쿼리 스트링은 숫자 10이 아니라 문자 10이다.
- @RequestParam을 사용하면 문자 10을 숫자 타입으로 받을 수 있다.
    - 스프링이 중간에서 타입을 변환해줬기 때문이다.

```text
@ModelAttribute UserData data

class UserData {
    Integer data;
}
```

```text
/users/{userId}

@PathVariable("data") Integer data
```

- 타입 변환은 @ModelAttribute와 @PathVariable도 마찬가지다.

### 타입 변환 예시

- 스프링 MVC 요청 파라미터를 사용할 때
    - @RequestParam, @ModelAttribute, @PathVariable
- @Value로 YML 정보를 읽을 때
- XML에 넣은 스프링 빈 정보를 변환할 때
- 뷰를 렌더링 할 때

## 스프링과 타입 변환

```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {
    T convert(S source);
}
```

- 스프링은 확장 가능한 컨버터 인터페이스를 제공한다.
    - 타입 변환이 필요하면 이 인터페이스를 구현해서 등록하면 된다.
- 모든 타입에 적용 가능하다.

### 참고

- 과거에는 PropertyEditor로 타입을 변환했다.
- 동시성 문제로 타입을 변환할 때마다 객체를 새로 생성해야 하는 문제가 있다.