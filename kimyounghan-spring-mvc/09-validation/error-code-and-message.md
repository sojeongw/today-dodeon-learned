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
    - proeprties의 {0}, {1}을 치환한다.