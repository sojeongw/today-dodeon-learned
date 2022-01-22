# Form 전송 객체 분리

실무에서는 등록할 때 Form에서 전달하는 데이터가 Item 도메인 객체와 딱 맞지 않는다. 따라서 Item 대신 별도의 객체를 만들어 전달한다.

```text
HTML Form -> Item -> Controller -> Item -> Repository
```

- 장점
    - Item 도메인 객체를 컨트롤러, 리파지토리까지 직접 전달한다.
    - 중간에 Item을 만드는 과정이 없어 간단하다.
- 단점
    - 간단한 경우에만 적용 가능하다.
    - 수정할 때 검증이 중복될 수 있어 groups를 사용해야 한다.

```text
HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository
```

- 장점
    - 전송하는 폼 데이터가 복잡해도 별도의 객체로 전달받을 수 있다.
    - 등록과 수정을 별도의 객체로 만들기 때문에 검증이 중복되지 않는다.
- 단점
    - Item 객체를 생성하는 변환 과정이 추가된다.

### 명명 규칙

ItemSave, ItemSaveForm, ItemSaveRequest, ItemSaveDto 등 의미있게 지으면 된다.

### 등록, 수정 뷰 템플릿이 비슷한데 통합이 가능한가?

어설프게 합치면 분기가 생겨서 나중에 유지보수할 때 고통스러울 수 있으므로 분리하는 게 좋다.

{% tabs %} {% tab title="ItemSaveForm.java" %}

```java

@Data
public class ItemSaveForm {

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(value = 9999)
    private Integer quantity;
}
```

{% endtab %} {% tab title="ItemUpdateForm.java" %}

```java

@Data
public class ItemUpdateForm {

    @NotNull
    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    // 수정에서는 수량을 자유롭게 변경할 수 있다.
    private Integer quantity;
}
```

{% endtab %} {% tab title="Item.java" %}

```java

@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

{% endtab %} {% endtabs %}

객체를 별도로 만들고 Item 객체는 원복한다.

{% tabs %} {% tab title="ValidationItemControllerV4.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v4/items")
@RequiredArgsConstructor
public class ValidationItemControllerV4 {

    private final ItemRepository itemRepository;

    @PostMapping("/add")
    // 뷰 템플릿과 이름을 맞추기 위해 name 옵션을 추가해준다.
    public String addItem(@Validated @ModelAttribute("item") ItemSaveForm itemSaveForm, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v4/addForm";
        }

        Item item = new Item();
        item.setItemName(itemSaveForm.getItemName());
        item.setPrice(itemSaveForm.getPrice());
        item.setQuantity(itemSaveForm.getQuantity());

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);

        return "redirect:/validation/v4/items/{itemId}";
    }

    @PostMapping("/{itemId}/edit")
    // 뷰 템플릿과 이름을 맞추기 위해 name 옵션을 추가해준다.
    public String edit(@PathVariable Long itemId, @Validated @ModelAttribute("item") ItemUpdateForm form, BindingResult bindingResult) {
        if (form.getPrice() != null && form.getQuantity() != null) {
            int resultPrice = form.getPrice() * form.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000,
                        resultPrice}, null);
            }
        }

        if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v4/editForm";
        }

        Item itemParam = new Item();
        itemParam.setItemName(form.getItemName());
        itemParam.setPrice(form.getPrice());
        itemParam.setQuantity(form.getQuantity());

        itemRepository.update(itemId, itemParam);
        return "redirect:/validation/v4/items/{itemId}";
    }
}
```

{% endtab %} {% endtabs %}

컨트롤러는 이제 새로운 객체를 요청으로 받고 저장할 때는 Item 객체로 변환한다.