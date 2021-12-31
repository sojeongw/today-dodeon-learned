# 오류 코드와 메시지 처리

## FieldError 생성자

- objectName
    - 오류가 발생한 객체 이름
- field
    - 오류가 발생한 필드
- rejectedValue
    - 사용자가 입력한 거절된 값
- bindingFailure
    - 타입 오류 등의 바인딩 실패인지, 검증 실패인지 구분하는 값
- codes
    - 메시지 코드
- arguments
    - 메시지에서 사용하는 인자
- defaultMessage
    - 기본 오류 메시지

이 중에서 codes와 argument을 이용해 에러 메시지를 작성해보자.

{% tabs %} {% tab title="application.properties" %}

```properties
# messages와 errors 두 곳 모두 찾아 쓰도록 설정한다.
# 생략하면 기본값인 messages만 읽어들인다.
spring.messages.basename=messages,errors
```

{% endtab %} {% tab title="errors.properties" %}

```properties
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
```

{% endtab %} {% tab title="ValidationItemControllerV2.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v2/items")
@RequiredArgsConstructor
public class ValidationItemControllerV2 {
    @PostMapping("/add")
    public String addItemV3(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("itemName", "itemName", item.getItemName(), false, new String[]{"required.item.itemName"}, null, null));
        }

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
            bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"}, new Object[]{100, 1_000_000}, null));
        }

        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, new String[]{"max.item.quantity"}, new Object[]{9_999}, null));
        }

        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();

            if (resultPrice < 10000) {
                // objectError는 기존 값이 있는 게 아니므로 fieldError 같은 생성자는 없다.
                bindingResult.addError(new ObjectError("item", new String[]{"totalPriceMin"}, new Object[]{10_000, resultPrice}, null));
            }
        }

        ...

        return "redirect:/validation/v2/items/{itemId}";
    }
}
```

{% endtab %} {% endtabs %}

기존 application.properties에 설정을 추가한다. 컨트롤러에는 이제 defaultMessage 파라미터를 넘기는 대신 code와 argument 파라미터에 데이터를 넘기면
errors.properties와 매핑된다.

- codes
    - 메시지 코드를 지정한다.
    - 배열이 들어가는 이유는 해당 키로 errors.properties에서 값을 찾지 못했을 때 다음 인덱스의 값으로 찾을 수 있기 때문이다.
- arguments
    - properties의 {0}, {1}을 치환한다.

## rejectValue(), reject()

```java

@Slf4j
@Controller
@RequestMapping("/validation/v2/items")
@RequiredArgsConstructor
public class ValidationItemControllerV2 {
    @PostMapping("/add")
    public String addItemV3(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    ...
    }
}
```

BindingResult는 항상 @ModelAttrubute 뒤에 온다. 사실 BindingResult는 이미 자기가 어떤 객체를 검증할지 알고 있다.

```java
log.info("objectName={}",bindingResult.getObjectName());
        log.info("target={}",bindingResult.getTarget());
```

```text
objectName=item // @ModelAttribute name
target=Item(id=null, itemName=상품, price=100, quantity=1234)
```

실제로 찍어보면 이렇게 된다.

{% tabs %} {% tab title="ValidationItemControllerV2.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v2/items")
@RequiredArgsConstructor
public class ValidationItemControllerV2 {

    @PostMapping("/add")
    public String addItemV4(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.rejectValue("itemName", "required");
        }

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
        }

        if (item.getQuantity() == null || item.getQuantity() > 10000) {
            bindingResult.rejectValue("quantity", "max", new Object[]{9999}, null);
        }

        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }

      ...
    }
}
```

{% endtab %} {% endtabs %}

이미 알고 있기 때문에 FeldError로 일일이 오브젝트 정보를 넣어주는 대신 rejectValue()와 reject()를 넣어준다.

### rejectValue()

- field
    - 오류 필드명
- errorCode
    - 오류 코드
    - FieldError와 다르게 앞글자만 입력해도 동작한다.
        - 메시지에 등록된 코드가 아니라 뒤에서 설명할 messageResolver를 위한 오류 코드다.
- errorArgs
    - 오류 메시지에서 {0}을 치환한다.
- defaultMessage
    - 오류 메시지를 찾을 수 없을 때 사용한다.

## 범용성 있는 오류 메시지 관리

- `required.item.itemName: 상품 이름은 필수입니다.`
    - 너무 자세하게 만들면 범용성이 떨어진다.
- `required: 필수 값입니다.`
    - 단순하게 만들면 여러 곳에서 사용할 수 있다.
    - 메시지를 세밀하게 작성하기 어렵다.

이런 문제는 범용적으로 사용하다가 세밀하게 작성하는 경우에는 세밀한 내용이 적용되도록 단계를 설정하는 방법으로 해결할 수 있다.

```text
# Level1
required.item.itemName: 상품 이름은 필수 입니다.

# Level2
required: 필수 값 입니다.
```

이렇게 상세한 메시지를 우선순위 높게 사용하는 것이다. 범용성 있게 잘 개발해두면 메시지를 추가해야할 때도 편리하게 관리할 수 있다.

## MessageCodesResolver

- 검증 오류 코드로 메시지 코드를 생성한다.
- MessageCodesResolver
    - 인터페이스
- DefaultMessageCodesResolver
    - 기본 구현체
- ObjectError, FieldError와 함께 사용한다.

```java
public class DefaultMessageCodesResolver implements MessageCodesResolver, Serializable {

    @Override
    public String[] resolveMessageCodes(String errorCode, String objectName) {
        return resolveMessageCodes(errorCode, objectName, "", null);
    }

    @Override
    public String[] resolveMessageCodes(String errorCode, String objectName, String field, @Nullable Class<?> fieldType) {
    ...
    }
}
```

MessageCodesResolver.resolveMessageCodes()는 errorCode objectName, field를 받는다.

```java
public class MessageCodesResolverTest {

    MessageCodesResolver messageCodesResolver = new DefaultMessageCodesResolver();

    @Test
    void messageCodesResolverObject() {
        // 에러 코드를 넣으면 메시지 코드가 여러개 나온다.
        String[] messageCodes = messageCodesResolver.resolveMessageCodes("required", "item");

        for (String messageCode : messageCodes) {
            System.out.println("messageCode = " + messageCode);
        }
    }
}
```

```text
messageCode = required.item
messageCode = required
```

실행해보면 errorCode와 objectName을 조합해 세세한 메시지 코드를 먼저 만들고 그 다음에 범용적인 코드를 만든다.

```java
public class MessageCodesResolverTest {

    MessageCodesResolver messageCodesResolver = new DefaultMessageCodesResolver();

    @Test
    void messageCodesResolverField() {
        String[] messageCodes = messageCodesResolver.resolveMessageCodes("required", "item", "itemName", String.class);

        assertThat(messageCodes).containsExactly(
                "required.item.itemName",
                "required.itemName",
                "required.java.lang.String",
                "required"
        );
    }
}
```

field를 넣으면 그것까지 포함해서 만들어준다. `required.java.lang.String`는 타입도 거를 수 있다.

## DefaultMessageCodesResolver의 기본 메시지 생성 규칙

### 객체 오류

1. code + "." + object name
2. code

`errorCode: required`, `object name: item`이라면

1. required.item
2. required

### 필드 오류

1. code + "." + object name + "." + field
2. code + "." + field
3. code + "." + field type
4. code

`errorCode: typeMismatch`, `object name: user`, `field: age`, `field type: int`이라면

1. typeMismatch.user.age
2. typeMismatch.age
3. typeMismatch.int
4. typeMismatch

## 동작 방식

- rejectValue(), reject()는 내부에서 MessageCodesResolver를 사용한다.
    - 여기서 메시지 코드를 생성한다.
- MessageCodesResolver를 통해 생성된 순서대로 오류 코드를 보관한다.
    - FieldError, ObjectError의 생성자를 보면 오류 코드는 여러 개를 가질 수 있다.

```text
codes [range.item.price, range.price, range.java.lang.Integer, range]
```

BindingResult 로그를 통해서도 확인할 수 있다. 

### FieldError

```java
rejectValue("itemName", "required")
```

- required.item.itemName
- required.itemName
- required.java.lang.String
- required

### ObjectError

```java
reject("totalPriceMin")
```

- totalPriceMin.item
- totalPriceMin