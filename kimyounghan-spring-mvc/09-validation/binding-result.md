# BindingResult

- 스프링이 제공하는 검증 오류를 보관한다.
- @ModelAttribute에 바인딩 하다가 에러가 나도 컨트롤러가 호출된다.
    - BindingResult가 없으면 타입 오류 발생 시 400 에러가 나면서 컨트롤러가 호출되지 않는다.
    - BindingResult가 있으면 타입 오류가 발생해도 오류 정보를 BindingResult에 담아 컨트롤러를 호출한다.
- 검증할 대상 바로 다음에 와야 한다.
    - ex. @ModelAttribute 바로 옆에 둬야 한다.
- Model에 자동으로 포함되어 뷰에 넘어간다.

## ValidationItemControllerV2

{% tabs %} {% tab title="ValidationItemControllerV2.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v2/items")
@RequiredArgsConstructor
public class ValidationItemControllerV2 {

    @PostMapping("/add")
    // BindingResult는 꼭 @ModelAttribute 옆에 있어야 한다.
    public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
        // 검증 로직
        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("itemName", "itemName", "상품 이름은 필수입니다."));
        }

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
            bindingResult.addError(new FieldError("item", "price", "가격은 1,000원에서 1,000,000원까지 허용합니다."));
        }

        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999까지 허용합니다."));
        }

        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();

            if (resultPrice < 10000) {
                bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
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

- BindingResult
    - @ModelAttribute 옆에 위치해야 어떤 데이터를 바인딩 하는지 인식할 수 있다.
- FieldError
    - 필드 오류를 담는다.
    - ObjectError의 자식이다.
- ObjectError
    - 글로벌 오류를 담는다.

### FieldError

```java
class FieldError extends ObjectError {
    public FieldError(String objectName, String field, String defaultMessage) {
    }
}
```

- objectName
    - @ModelAttribute 이름
- field
    - 오류가 발생한 필드 이름
- defaultMessage
    - 오류 기본 메시지

### ObjectError

```java
class ObjectError {
    public ObjectError(String objectName, String defaultMessage) {
    }
}
```

- objectName
    - @ModelAttribute 이름
- defaultMessage
    - 오류 기본 메시지

## BindingResult에 검증 오류를 적용하는 3가지 방법

1. @ModelAttribute에 타입 요류 등 바인딩이 실패하는 경우 스프링이 FieldError를 생성해 BindingResult에 넣는다.
2. 개발자가 직접 코드 상으로 넣어준다.
3. Validator를 사용한다.

## BindingResult와 Errors

![](../../.gitbook/assets/kimyounghan-spring-mvc/09/screenshot%202022-03-09%20오후%207.00.47.png)

- BindingResult 인터페이스는 Errors 인터페이스를 상속한다.
    - 인터페이스가 인터페이스를 받을 때는 extends를 쓴다.
- 실제 구현체는 BeanPropertyBindingResult다.
    - BindingResult, Errors 둘 다 구현하고 있기 때문에 실제 컨트롤러에 코드를 작성할 때는 Errors를 사용해도 된다.
- Errors는 오류 단순 저장과 조회만 제공하기 때문에 기능이 별로 없다.
    - 그래서 추가 기능을 제공하는 BindingResult를 많이 사용한다.
