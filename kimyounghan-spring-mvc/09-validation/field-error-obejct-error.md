# FieldError, ObjectError

사용자가 입력한 값이 오류 메시지가 발생해도 그대로 남아있도록 해보자.

![](../../.gitbook/assets/kimyounghan-spring-mvc/09/screenshot%202022-03-09%20오후%207.12.04.png)

FieldError는 2가지 생성자를 지원한다.

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

{% tabs %} {% tab title="ValidationItemControllerV2.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v2/items")
@RequiredArgsConstructor
public class ValidationItemControllerV2 {
    
    @PostMapping("/add")
    public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("itemName", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수입니다."));
        }

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
            bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, null, null, "가격은 1,000원에서 1,000,000원까지 허용합니다."));
        }

        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, null, null, "수량은 최대 9,999까지 허용합니다."));
        }

        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();

            if (resultPrice < 10000) {
                // objectError는 기존 값이 있는 게 아니므로 fieldError 같은 생성자는 없다.
                bindingResult.addError(new ObjectError("item", null, null, "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
            }
        }

        if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v2/addForm";
        }

        // 성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);

        return "redirect:/validation/v2/items/{itemId}";
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/09/screenshot%202022-03-09%20오후%207.17.03.png)

이제 에러가 발생해도 잘못 입력한 값이 폼에 그대로 남아있다.

사용자의 입력 데이터가 컨트롤러의 @ModelAttribute에 바인딩 되는 시점에 오류가 발생하면 모델 객체에 입력했던 값을 유지하기가 힘들다. Integer 타입에 문자를 입력한다면 보관할 수 있는 방법이 없다. 따라서 오류가 발생하면 FieldError를 이용해 입력 값을 별도로 저장하고 화면에 다시 출력한다.

스프링은 타입 오류로 바인딩에 실패하면 FieldError를 생성하면서 사용자가 입력한 오류를 넣어둔다. 그리고 해당 오류를 BindingResult에 담아 컨트롤러를 호출한다.