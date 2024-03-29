# 스프링 MVC 웹 페이지 만들기

## 상품 도메인 개발

{% tabs %} {% tab title="BasicItemController.java" %}

```java

@Controller
@RequiredArgsConstructor
@RequestMapping("/basic/items")
public class BasicItemController {

    private final ItemRepository itemRepository;

    /*
    생성자 주입으로 빈을 가져온다. 생성자가 하나만 있으면 @Autowired를 생략할 수 있다.
    여기서 롬복 @RequiredArgsConstructor까지 쓰면 코드 모두 생략 가능하다.

    @Autowired
    public BasicItemController(ItemRepository itemRepository) {
        this.itemRepository = itemRepository;
    }
    */

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);

        return "basic/items";
    }

    // 테스트용 데이터 추가
    @PostConstruct
    public void init() {
        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));
    }
}
```

{% endtab %} {% tab title="Item.java" %}

```java
// 실무에서는 도메인에 @Data를 쓰면 위험하다.
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

{% endtab %} {% tab title="ItemRepository.java" %}

```java

@Repository
public class ItemRepository {

    // 실무에서는 멀티 스레드이기 때문에 HashMap 대신 ConcurrentHashMap을 써야 한다.
    private static final Map<Long, Item> store = new HashMap<>();
    public static long sequence = 0L;

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }

    public Item findById(Long id) {
        return store.get(id);
    }

    public List<Item> findAll() {
        // 컬렉션으로 한 번 감싸서 반환하면 store 자체 값에는 영향을 미치지 않는다.
        return new ArrayList<>(store.values());
    }

    // 정석대로라면 프로퍼티에 id를 제외한 즉, 필요한 파라미터만 있는 객체를 따로 사용하는 게 좋다. 
    // 안쓰는 프로퍼티가 있으면 개발자가 헷갈릴 수 있기 때문이다.
    public void update(Long itemId, Item updateParam) {
        Item findItem = findById(itemId);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    public void clearStore() {
        store.clear();
    }
}
```

{% endtab %} {% endtabs %}

## 상품 등록 처리

### addItemV1

{% tabs %} {% tab title="BasicItemController.java" %}

```java

@Controller
@RequiredArgsConstructor
@RequestMapping("/basic/items")
public class BasicItemController {

    private final ItemRepository itemRepository;

    ...

    @PostMapping("/add")
    public String addItemV1(@RequestParam String itemName, @RequestParam int price, @RequestParam Integer quantity, Model model) {
        Item item = new Item();
        item.setItemName(itemName);
        item.setPrice(price);
        item.setQuantity(quantity);

        itemRepository.save(item);

        // 새로 저장한 아이템에 대한 새 페이지를 만드는 대신, 템플릿에 값을 넘긴다.
        model.addAttribute(item);
        return "basic/item";
    }
    
    ...
}

```

{% endtab %} {% endtabs %}

- 요청 파라미터 데이터를 하나하나 변수로 받는다.
- ItemRepository에 저장한 뒤 item을 모델에 담아 뷰에 전달한다.

### addItemV2

{% tabs %} {% tab title="BasicItemController.java" %}

```java

@Controller
@RequiredArgsConstructor
@RequestMapping("/basic/items")
public class BasicItemController {

    private final ItemRepository itemRepository;

    ...

    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item, Model model) {
        /*
        modelAttribute가 item을 V1이랑 똑같이 만들어준다.
        Item item = new Item();
        item.setItemName(itemName);
        item.setPrice(price);
        item.setQuantity(quantity);
        */

        itemRepository.save(item);

        /*
        @ModelAttribute("item")에 지정한 item 이란 이름으로 뷰에도 담아준다.
        모델에 들어가는 값이 일치하지 않으면 안된다.
        예를 들어 템플릿에 item이라고 되어있는데 @ModelAttribute("item2")라고 하면 실패한다.
        model.addAttribute(item);
        */

        return "basic/item";
    }
    
    ...
}
```

{% endtab %} {% endtabs %}

- @ModelAttribute가 Item 객체를 생성하고 모델에 넣어주는 일을 대신 해준다.
- 모델에 데이터를 담을 때는 이름이 필요한데 이때 애너테이션 안에 name 속성을 넣어주면 된다.
    - 이름 지정
        - `@ModelAttribute("hello") Item item`
    - 모델에 해당 이름으로 저장
        - `model.addAttribute("hello", item);`

### addItemV3

{% tabs %} {% tab title="BasicItemController.java" %}

```java

@Controller
@RequiredArgsConstructor
@RequestMapping("/basic/items")
public class BasicItemController {

    private final ItemRepository itemRepository;

    ...

    @PostMapping("/add")
    public String addItemV3(@ModelAttribute Item item) {
        itemRepository.save(item);
        return "basic/item";
    }
    
    ...
}
```

{% endtab %} {% endtabs %}

- 이름을 생략하면 클래스 이름의 맨 앞글자만 소문자로 바꾼 것이 이름이 된다.
    - `HelloData` -> `helloData`

### addItemV4

{% tabs %} {% tab title="BasicItemController.java" %}

```java

@Controller
@RequiredArgsConstructor
@RequestMapping("/basic/items")
public class BasicItemController {

    private final ItemRepository itemRepository;

    ...

    @PostMapping("/add")
    public String addItemV4(Item item) {
        itemRepository.save(item);
        return "basic/item";
    }
    
    ...
}
```

{% endtab %} {% endtabs %}

- @ModelAttribute를 완전히 생략해도 그대로 모델에 자동 등록된다. 동작 방식도 같다.
    - 단순 타입은 @RequestParam, 임의의 객체는 @ModelAttribute가 적용된다.

## 상품 수정

{% tabs %} {% tab title="BasicItemController.java" %}

```java

@Controller
@RequiredArgsConstructor
@RequestMapping("/basic/items")
public class BasicItemController {

    private final ItemRepository itemRepository;

    ...

    @GetMapping("/{itemId}/edit")
    public String editForm(@PathVariable Long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);

        return "basic/editForm";
    }

    @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
        itemRepository.update(itemId, item);
        return "redirect:/basic/items/{itemId}";
    }
    
    ...
}
```

{% endtab %} {% endtabs %}

- @GetMapping("/{itemId}/edit")
    - 상품 수정 폼으로 이동
- @PostMapping("/{itemId}/edit")
    - 상품 수정 로직 처리
- redirect:/basic/items/{itemId}
    - 뷰 템플릿 호출 대신, 상품 상세 화면으로 이동하도록 리다이렉트 한다.
    - 컨트롤러에 매핑된 @PathVariable 값도 사용할 수 있다.

### HTML Form 전송

- GET, POST만 지원한다.
    - PUT, PATCH는 HTTP API 전송 시에 사용한다.
    - HTTP POST로 Form 요청 시 히든 필드를 통해 PUT, PATCH를 사용할 수도 있지만 HTTP 요청 상으로는 POST다.

## PRG Post/Redirect/Get

![](../../.gitbook/assets/kimyounghan-spring-mvc/07/screenshot%202022-03-05%20오후%201.15.40.png)

상픔 등록 폼에서 상품 등록을 요청하면 상품 상세 페이지가 나오도록 개발했다.

사실 이 애플리케이션엔 상품 등록 폼에서 상품 등록을 한 뒤 새로고침을 하면 계속 상품이 생성되는 버그가 있다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/07/screenshot%202022-03-05%20오후%201.16.23.png)

웹 브라우저의 새로고침은 마지막에 서버에 전송한 데이터를 다시 전송한다는 의미다.

따라서 상품 등록을 요청한 뒤 새로고침을 누르면 다시 같은 데이터를 서버로 전송하게 된다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/07/screenshot%202022-03-05%20오후%201.21.38.png)

확인하면 똑같은 데이터로 POST 요청을 하고 있음을 알 수 있다.

### 해결 방법

![](../../.gitbook/assets/kimyounghan-spring-mvc/07/screenshot%202022-03-05%20오후%201.16.41.png)

저장 후 리다이렉트를 하면 웹 브라우저 입장에서는 완전히 다른 url을 받게 된다. 그럼 새로고침을 해도 직전에 요청한 GET의 결과를 받게 된다. 이 방식을 PRG라고 부른다.

{% tabs %} {% tab title="addItemV5" %}

```java

@Controller
@RequiredArgsConstructor
@RequestMapping("/basic/items")
public class BasicItemController {

    private final ItemRepository itemRepository;

    ...

    @PostMapping("/add")
    public String addItemV5(Item item) {
        itemRepository.save(item);

        // redirect로 바꾼다.
        return "redirect:/basic/items/" + item.getId();
    }
    
    ...
}
```

{% endtab %} {% tab title="addItemV4" %}

```java

@Controller
@RequiredArgsConstructor
@RequestMapping("/basic/items")
public class BasicItemController {

    private final ItemRepository itemRepository;

    ...

    @PostMapping("/add")
    public String addItemV4(Item item) {
        itemRepository.save(item);
        return "basic/item";
    }
    
    ...
}
```

{% endtab %} {% endtabs %}

따라서 상품 등록 후 상품 상세로 가도록 redirect 하면 문제가 해결된다.

### 주의 사항

`+ item.getId()`처럼 URL에 변수를 더해서 넣으면 URL 인코딩이 안되기 때문에 위험하다. 다음에 나올 RedirectAttributes를 사용하면 해결된다.

## RedirectAttributes

고객 입장에서 등록이 잘 된건지 확인이 어려우니 개선해보자.

{% tabs %} {% tab title="BasicItemController.java" %}

```java

@Controller
@RequiredArgsConstructor
@RequestMapping("/basic/items")
public class BasicItemController {

    private final ItemRepository itemRepository;

    ...

    @PostMapping("/add")
    public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
        Item savedItem = itemRepository.save(item);

        // 여기 적힌 attributeName인 itemId가
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);

        // 이곳 url로 들어온다.
        return "redirect:/basic/items/{itemId}";
    }
    
    ...
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/07/screenshot%202022-03-05%20오후%201.41.40.png)

itemId와 status가 전달된 걸 확인할 수 있다.

{% tabs %} {% tab title="item.html" %}

```html
...

<div class="container">
    <div class="py-5 text-center">
        <h2>상품 상세</h2></div>
    <div>
        <!-- 추가 -->
        <h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>

        ...
    </div>
</div>

...
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-mvc/07/screenshot%202022-03-05%20오후%201.43.29.png)

status가 true면 텍스트를 띄우도록 설정하면 등록이 확실하게 됐다는 걸 알 수 있다.
