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

## WebDataBinder

{% tabs %} {% tab title="ValidationItemControllerV2.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v2/items")
@RequiredArgsConstructor
public class ValidationItemControllerV2 {

    private final ItemRepository itemRepository;
    private final ItemValidator itemValidator;

    @InitBinder
    public void init(WebDataBinder webDataBinder) {
        webDataBinder.addValidators(itemValidator);
    }

    @PostMapping("/add")
    // @Validated 애너테이션을 추가한다.
    public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
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

@InitBinder가 설정된 컨트롤러의 어떤 메서드든 호출될 때마다 dataBinder가 항상 새롭게 만들어져 적용된다. validator 코드를 삭제하고 애너테이션만 달아도 검증이 작동한다.

### 동작 방식

- @Validator 애너테이션이 붙으면 WebDataBinder에 등록한 검증기를 찾아 실행한다.
- 여러 검증기를 등록한다면 supports()를 사용해 어떤 검증기가 실행될지 확인한다.
    - 앞서 정의한 supports()를 호출하면 Item 타입인지 확인한다.
    - 결과가 true이므로 ItemValidator의 validate()가 호출된다.

### 글로벌 설정

```java

@SpringBootApplication
public class ItemServiceApplication implements WebMvcConfigurer {

    public static void main(String[] args) {
        SpringApplication.run(ItemServiceApplication.class, args);
    }

    @Override
    public Validator getValidator() {
        return new ItemValidator();
    }
}
```

모든 컨트롤러에 다 적용하고 싶다면 글로벌 설정이 가능하다. 따로 정의된 게 있다면 supports()가 getValidator()보다 먼저 호출된다.

### 참고

- @Validated
    - 스프링 전용 검증 애너테이션
- @Valid
    - 자바 표준 검증 애너테이션

@Validated, @Valid 모두 검증할 때 사용 가능하다. 하지만 후자는 `org.springframework.boot:spring-boot-starter-validation` 의존성이 필요하다.