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