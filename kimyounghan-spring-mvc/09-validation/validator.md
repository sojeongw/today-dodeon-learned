# Validator 분리

{% tabs %} {% tab title="ItemValidator.java" %}

```java

@Component
public class ItemValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        // Item 및 Item의 자식의 타입인지 확인한다.
        // 자식까지 확인하기 위해 isAssignableFrom()을 쓴다.
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        // Object 타입이기 때문에 캐스팅을 해야 한다.
        Item item = (Item) target;

        if (!StringUtils.hasText(item.getItemName())) {
            // Errors의 자식이 BidingResult라서 담을 수 있다.
            errors.rejectValue("itemName", "required");
        }

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            errors.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
        }

        if (item.getQuantity() == null || item.getQuantity() > 10000) {
            errors.rejectValue("quantity", "max", new Object[]{9999}, null);
        }

        // 특정 필드 예외가 아닌 전체 예외
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }
    }
}
```

{% endtab %} {% tab title="ValidationItemControllerV2.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v2/items")
@RequiredArgsConstructor
public class ValidationItemControllerV2 {

    private final ItemRepository itemRepository;
    // 추가하면서 파라미터는 두 개가 됐지만 생성자는 하나니까 @Autowired가 붙은 것처럼 자동으로 주입해준다.
    private final ItemValidator itemValidator;

    @PostMapping("/add")
    public String addItemV5(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
        itemValidator.validate(item, bindingResult);

        if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v2/addForm";
        }

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);

        return "redirect:/validation/v2/items/{itemId}";
    }
}
```

{% endtab %} {% endtabs %}

검증 로직을 분리해 컨트롤러의 코드를 깔끔하게 유지할 수 있다.

- supports()
    - 검증기를 지원하는지 확인한다.
- validate(Object target, Errors errors)
    - 검증 대상 객체와 BindingResult를 파라미터로 받는다.