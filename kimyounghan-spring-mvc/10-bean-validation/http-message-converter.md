# HTTP 메시지 컨버터

@Valid, @Validated는 HttpMessageConverter를 사용하년 @RequestBody에도 적용할 수 있다.

- @ModelAttribute
    - HTTP 요청 파라미터인 URL 쿼리 스트링, POST Form 데이터를 다룰 때 사용한다.
- @RequestBody
    - HTTP Body의 데이터를 객체로 변환할 때 사용한다.
    - 주로 API JSON 요청 시 사용한다.

```java

@Slf4j
@RestController
@RequestMapping("/validation/api/items")
public class ValidationItemApiController {

    @PostMapping("/add")
    public Object addItem(@RequestBody @Validated ItemSaveForm form, BindingResult bindingResult) {
        log.info("API 컨트롤러 호출");

        if (bindingResult.hasErrors()) {
            log.info("검증 오류 발생 errors={}", bindingResult);
            return bindingResult.getAllErrors();
        }

        log.info("성공");
        return form;
    }
}
```

### 성공

```text
API 컨트롤러 호출
성공
```

### 실패

```text
POST http://localhost:8080/validation/api/items/add
{"itemName":"hello", "price":"A", "quantity": 10}
```

```json
{
  "timestamp": "2021-04-20T00:00:00.000+00:00",
  "status": 400,
  "error": "Bad Request",
  "message": "",
  "path": "/validation/api/items/add"
}
```

```text
.w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.http.converter.HttpMessageNotReadableException: 
JSON parse error: Cannot deserialize value of type `java.lang.Integer` from String "A":not a valid Integer value; 
nested exception is com.fasterxml.jackson.databind.exc.InvalidFormatException: Cannot deserialize value of type `java.lang.Integer` from String "A": 
not a valid Integer value at [Source: (PushbackInputStream); line: 1, column: 30] (through reference chain: hello.itemservice.domain.item.Item["price"])]
```

HttpMessageConverter에서 JSON을 Item 객체로 변환하는 걸 실패했다. 컨트롤러 자체를 호출하지 못하기 때문에 Validator도 실행되지 않는다.

```text
POST http://localhost:8080/validation/api/items/add
{"itemName":"hello", "price":1000, "quantity": 10000}
```

```json
[
  {
    "codes": [
      "Max.itemSaveForm.quantity",
      "Max.quantity",
      "Max.java.lang.Integer",
      "Max"
    ],
    "arguments": [
      {
        "codes": [
          "itemSaveForm.quantity",
          "quantity"
        ],
        "arguments": null,
        "defaultMessage": "quantity",
        "code": "quantity"
      },
      9999
    ],
    "defaultMessage": "9999 이하여야 합니다",
    "objectName": "itemSaveForm",
    "field": "quantity",
    "rejectedValue": 10000,
    "bindingFailure": false,
    "code": "Max"
  }
]
```

```text
API 컨트롤러 호출
검증 오류 발생, errors=org.springframework.validation.BeanPropertyBindingResult: 1 errors
Field error in object 'itemSaveForm' on field 'quantity': rejected value[99999]; codes
[Max.itemSaveForm.quantity,Max.quantity,Max.java.lang.Integer,Max]; arguments
[org.springframework.context.support.DefaultMessageSourceResolvable: codes
[itemSaveForm.quantity,quantity]; arguments []; default message
[quantity],9999]; default message [9999 이하여야 합니다]
```

`bindingResult.getAllErrors();` 코드가 ObjectError와 FieldError를 반환하면 스프링은 이 객체를 JSON으로 변환해 클라이언트에 전달한다. 실무에서는 이 중 필요한 데이터만
뽑아 별도의 객체로 반환한다.

로그를 보면 검증 오류가 정상적으로 수행되었다.

## @ModelAttribute vs @RequestBody

### @ModelAttribute

- 각 필드 단위로 세밀하게 적용된다.
- 특정 필드의 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상적으로 처리할 수 있다.
- 따라서 Validator로 검증도 가능하다.

### @RequestBody(HttpMessageConverter)

- 각 필드 단위가 아니라 전체 객체 단위로 적용된다.
- 메시지 컨버터가 성공해서 객체를 만들어야 @Valid, @Validated가 적용된다.
- 메시지 컨버터에서 실패하면 이후 단계가 진행되지 않고 예외가 발생한다.
    - 컨트롤러가 호출되지 않으며 Validator도 적용할 수 없다.