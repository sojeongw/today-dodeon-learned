# Bean Validation

- 검증 애너테이션과 인터페이스의 모음
- Bean Validation 2.0(JSR-380)이라는 기술 표준
    - 특정 구현체가 아니다.
- 대표적인 구현체는 하이버네이터 Validator가 있다.
    - 이름은 하이버네이트이지만 ORM과는 관련이 없다.

[하이버네이트 Validator](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/)

## 의존 관계 추가

```groovy
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

## 테스트 코드 작성

```java
import lombok.Data;
// 하이버네이트 validator 구현체를 사용할 때만 제공된다. 실무에서 대부분 사용하므로 자유롭게 쓰면 된다.
import org.hibernate.validator.constraints.Range;

// 특정 구현에 관계없이 제공되는 표준 인터페이스
import javax.validation.constraints.Max;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

@Data
public class Item {

    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1_000_000)
    private Integer price;

    @NotNull
    @Max(9999)
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

```java
public class BeanValidationTest {

    @Test
    void beanValidation() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        // 검증기 생성
        Validator validator = factory.getValidator();

        Item item = new Item();
        item.setItemName(" ");
        item.setPrice(0);
        item.setQuantity(10000);

        Set<ConstraintViolation<Item>> violations = validator.validate(item);

        for (ConstraintViolation<Item> violation : violations) {
            System.out.println("violation=" + violation);
            System.out.println("violation.message=" + violation.getMessage());
        }
    }
}

```

```text
violation={interpolatedMessage='공백일 수 없습니다', propertyPath=itemName,rootBeanClass=class hello.itemservice.domain.item.Item,messageTemplate='{javax.validation.constraints.NotBlank.message}'}
violation.message=공백일 수 없습니다

violation={interpolatedMessage='9999 이하여야 합니다', propertyPath=quantity,rootBeanClass=class hello.itemservice.domain.item.Item,messageTemplate='{javax.validation.constraints.Max.message}'}
violation.message=9999 이하여야 합니다

violation={interpolatedMessage='1000에서 1000000 사이여야 합니다',propertyPath=price, rootBeanClass=class hello.itemservice.domain.item.Item,messageTemplate='{org.hibernate.validator.constraints.Range.message}'}
violation.message=1000에서 1000000 사이여야 합니다
```

애너테이션 하나로 간단하게 검증할 수 있다. 스프링은 빈 검증기를 통합해뒀기 때문에 실제로 직접 빈 검증기를 쓸 일은 없다.

## 스프링 적용

{% tabs %} {% tab title="ValidationItemControllerV3.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v3/items")
@RequiredArgsConstructor
public class ValidationItemControllerV3 {

    @PostMapping("/add")
    public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v3/addForm";
        }

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);

        return "redirect:/validation/v3/items/{itemId}";
    }
}
```

{% endtab %} {% endtabs %}

기존 Validator 코드를 제거해도 실행해보면 검증기가 적용된다. validation 의존성을 추가하면 스프링이 자동으로 처리해주기 때문이다.

### 작동 원리

1. LocalValidatorFactoryBean을 글로벌 Validator로 등록한다.
2. 애너테이션을 보고 검증을 수행한다.

따라서 @Valid, Validated 애너테이션만 적용하면 된다. 검증 오류가 발생하면 FieldError, ObjectError를 생성해 BindingResult에 담는다.

만약 이전에 만들었던 글로벌 검증기를 적용하고 있다면 스프링 부트가 자동으로 Validator를 등록하지 않기 때문에 제거해준다.

### 검증 순서

1. @ModelAttribute가 각 필드에 타입 변환을 시도한다.
    - requestParam을 각 필드에 넣어준다.
2. 성공하면 Validatior를 적용한다.
3. 실패하면 typeMismatch로 FieldError를 추가한다.
    - Validator는 적용하지 않는다.

즉, 바인딩에 성공한 필드만 Bean Validation을 적용한다. 일단 모델 객체에서 바인딩 받는 값이 정상으로 들어와야 검증도 의미가 있기 때문이다.

## 에러 코드

각 애너테이션은 그 이름을 오류 코드로 해서 메시지 코드를 생성한다.

### @NotBlank

- NotBlank.item.itemName
- NotBlank.itemName
- NotBlank.java.lang.String
- NotBlank

### @Range

- Range.item.price
- Range.price
- Range.java.lang.Integer
- Range

따라서 메시지를 등록하면 Bean Validation의 기본 기능의 메시지를 수정할 수 있다.

{% tabs %} {% tab title="errors.properties" %}

```properties
...
# Bean Validation 추가
NotBlank={0} 공백X
Range={0}, {2} ~ {1} 허용
Max={0}, 최대 {1}
```

{% endtab %} {% endtabs %}

## 메시지 검색 우선순위

1. 생성된 메시지 코드의 순서대로 messageSource에서 메시지 검색
2. 애너테이션에 정의한 message 속성
    - @NotBlank(message = "공백은 넣을 수 없습니다. {0}")
3. 라이브러리가 제공하는 기본값

## 오브젝트 오류

### @ScriptAssert

Bean Validation 애너테이션은 필드에 선언한다. 오브젝트 에러를 만들고 싶다면 어떻게 해야할까?

{% tabs %} {% tab title="Item.java" %}

```java

@Data
@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000")
public class Item {

    ...
}

```

{% endtab %} {% endtabs %}

```text
스크립트 표현식 "_this.price * _this.quantity >= 10000"가 true로 평가되지 않았습니다
상품의 가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = 1,000
```

- 메시지 코드
    - ScriptAssert.item
    - ScriptAssert

이 방식은 제약 조건이 많고 복잡하다. 실무에서 나오는 다양한 조건을 만족하기도 어렵다.

### 자바 코드 작성

```java

@Slf4j
@Controller
@RequestMapping("/validation/v3/items")
@RequiredArgsConstructor
public class ValidationItemControllerV3 {

    @PostMapping("/add")
    public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
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

따라서 그냥 자바 코드로 풀어내는 것을 권장한다.

## Groups

데이터 등록과 수정의 요구 사항이 다를 수 있다. 만약 수정 기능에선 id가 @NotNull이라면 같은 객체를 사용하는 등록 기능에 문제가 발생한다.

그래서 Bean Validation은 groups라는 기능을 제공한다. 예를 들어 등록과 수정을 다른 그룹으로 나눠 적용할 수 있다.

{% tabs %} {% tab title="SaveCheck.java" %}

```java
public interface SaveCheck {
}
```

{% endtab %} {% tab title="UpdateCheck.java" %}

```java
public interface UpdateCheck {
}
```

{% endtab %} {% endtabs %}

{% tabs %} {% tab title="Item.java" %}

```java

@Data
public class Item {

    // 수정할 때만 적용한다.
    @NotNull(groups = UpdateCheck.class)
    private Long id;

    @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
    private String itemName;

    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
    private Integer price;

    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    // 등록할 때만 적용한다.
    @Max(value = 9999, groups = SaveCheck.class)
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

{% endtab %} {% tab title="ValidationItemControllerV3.java" %}

```java

@Slf4j
@Controller
@RequestMapping("/validation/v3/items")
@RequiredArgsConstructor
public class ValidationItemControllerV3 {

    @PostMapping("/add")
    // 각 메서드에 적용한다.
    public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
        ...
    }
}
```

{% endtab %} {% endtabs %}

@Validated에 groups 기능을 적용하면 된다. @Valid에는 이 기능이 없다.

하지만 복잡도가 올라가고 실무에서는 다른 방법을 쓰기 때문에 잘 쓰지 않는다.