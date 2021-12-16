# Validation

폼에 잘못된 값을 입력하면 친절하게 알려줘야 한다. HTTP 요청이 정상인지 검증하는 것도 컨트롤러의 중요한 역할 중 하나다.

- 클라이언트 검증은 조작할 수 있어 보안에 취약하다.
    - 자바스크립트로 검증 로직을 잘 만들어도 포스트맨으로 값 바꿔서 넘겨버릴 수도 있다.
- 그렇다고 서버만으로 검증하면 고객 사용성이 좋지 않다.
    - 클라이언트에서 창을 바로 띄워주면 사용성이 좋아질 수 있다.
- 둘을 적절히 섞어 사용하되 최종적으로 서버 검증은 필수다.
- API 방식을 사용하면 API 스펙을 잘 정의해 검증 오류를 API 응답 결과에 잘 남겨줘야 한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/09/screenshot%202022-03-09%20오후%203.30.48.png)

폼에 정상 데이터를 입력하면 서버에서는 검증 로직을 통과하고, 상품을 저장하고, 상세 화면으로 redirect 한다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/09/screenshot%202022-03-09%20오후%203.31.20.png)

폼에 값을 상품명을 입력하지 않거나, 가격이나 수량이 너무 작거나 크면 서버 검증 로직이 실패해야 한다.

이렇게 검증에 실패한 경우 고객에게 다시 상품 등록 폼을 보여주고 어떤 값이 잘못됐는지 알려줘야 한다.

## ValidationItemControllerV1

{% tabs %} {% tab title="ValidationItemControllerV1.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v1/items")
@RequiredArgsConstructor
public class ValidationItemControllerV1 {

    @PostMapping("/add")
    public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {
        // 검증 오류를 결과를 보관하는 객체를 만든다.
        Map<String, String> errors = new HashMap<>();

        // 검증 로직
        if (!StringUtils.hasText(item.getItemName())) {
            errors.put("itemName", "상품 이름은 필수입니다.");
        }

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
            errors.put("price", "가격은 1,000원에서 1,000,000원까지 허용합니다.");
        }

        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            errors.put("quantity", "수량은 최대 9,999까지 허용합니다.");
        }

        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();

            if (resultPrice < 10000) {
                // 여러 필드에 대한 오류는 필드 이름을 넣을 수 없으니 globalError로 넣는다.
                errors.put("globalError", "가격 * 수량은 10,000원 이상이어야 합니다.");
            }
        }

        // 에러가 있다면 입력 폼으로 다시 돌아간다.
        if (!errors.isEmpty()) {
            log.info("errors = {}", errors);
            model.addAttribute("errors", errors);
            return "validation/v1/addForm";
        }

        // 성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);

        return "redirect:/validation/v1/items/{itemId}";
    }
}
```

{% endtab %} {% endtabs %}

- 검증 로직을 구현 후 에러가 있으면 Map에 담고 입력 폼으로 다시 돌려보낸다.

### 남은 문제점

- 뷰 템플릿에 중복 처리가 많다.
- 타입 오류 처리가 안된다.
    - 숫자 타입에 문자가 들어오면 오류가 발생한다.
    - 이 오류는 스프링 MVC에서 컨트롤러에 들어오기도 전에 발생하는 예외이기 때문에 컨트롤러가 호출되지 않고 400 에러가 발생한다.
- 따라서 고객이 입력한 값도 별도로 관리가 되어야 한다.
    - 만약 컨트롤러가 호출되더라도 Integer이므로 문자를 보관할 수 없다.
    - 바인딩이 불가하므로 입력한 문자가 사라지고 어떤 내용을 입력해서 오류가 발생했는지 고객 입장에서 이해하기 어렵다.

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