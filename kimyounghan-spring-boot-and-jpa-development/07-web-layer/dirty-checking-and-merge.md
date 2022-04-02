# 변경 감지와 병합

## 준영속 Entity

- JPA 영속성 컨텍스트가 더 이상 관리하지 않는 Entity

```java

@Controller
@RequiredArgsConstructor
public class ItemController {

  @PostMapping(value = "/items/{itemId}/edit")
  public String updateItem(@ModelAttribute("form") BookForm form) {
    Book book = new Book();

    book.setId(form.getId());
    book.setName(form.getName());
    book.setPrice(form.getPrice());
    book.setStockQuantity(form.getStockQuantity());
    book.setAuthor(form.getAuthor());
    book.setIsbn(form.getIsbn());

    itemService.saveItem(book);

    return "redirect:/items";
  }
}
```

`form.getId()`를 보면 어디선가 DB에 한 번 갔다가 받아온 데이터인 걸 알 수 있다. 이미 DB에 한 번 저장되어 식별자가 있는 데이터가 준영속 Entity다.

영속 상태 Entity는 JPA가 관리하기 때문에 더티 체킹이 일어나지만 이준영속 Entity는 JPA가 관리하지 않으므로 DB 업데이트가 안 일어난다.

준영속 Entity는 변경 감지 기능과 병합 기능으로 수정할 수 있다.

## 변경 감지 기능

```java

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class ItemService {

  @Transactional
  public void updateItem(Long itemId, Item param) {
    Item findItem = itemRepository.findOne(itemId);
    findItem.setPrice(param.getPrice());
    findItem.setName(param.getName());
  }
}
```

1. 트랜잭션 안에서 Entity를 다시 조회한다.
2. 변경할 값을 선택한다.
3. 트랜잭션 커밋 시점에 변경 감지(Dirty Checking)가 일어난다.
4. 데이터베이스에 UPDATE SQL 실행된다.

준영속 Entity 값으로 영속 상태 Entity에 세팅하면 된다. 영속 상태 Entity는 더티체킹 덕분에 save()할 필요가 없다.

## 병합 기능

```java

@Repository
@RequiredArgsConstructor
public class ItemRepository {

  private final EntityManager em;

  @Transactional
  public void update(Item item) {
    Item mergedItem = em.merge(item);
  }

}
```

1. 준영속 Entity의 식별자 값으로 영속 Entity를 조회한다.
2. 영속 Entity의 값을 준영속 Entity의 값으로 모두 교체(병합)한다.
3. 트랜잭션 커밋 시점에 변경 감지 기능이 동작해서 데이터베이스에 UPDATE SQL이 실행된다.

영속성 컨텍스트에서 똑같은 식별자를 가진 데이터를 찾고, 파라미터 값으로 모든 값을 바꿔치기 한다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/07/screenshot%202021-05-22%20오후%207.50.15.png)

1. merge()를 실행한다.
2. 파라미터로 넘어온 준영속 Entity의 식별자 값으로 1차 캐시에서 Entity를 조회한다.
    - 만약 1차 캐시에 Entity가 없으면 데이터베이스에서 Entity를 조회하고, 1차 캐시에 저장한다.
3. 조회한 영속 Entity mergeMember에 member Entity의 값을 채워 넣는다.
    - member Entity의 모든 값을 mergeMember에 밀어 넣는다.
    - 이때 mergeMember의 `회원1`이라는 이름이 `회원명변경`으로 바뀐다.
4. 영속 상태인 mergeMember를 반환한다.

주의할 점은 파라미터로 넘어온 item은 영속 Entity로 안 바뀐다는 것이다. 병합이 된 mergedItem만 영속성 컨텍스트에서 관리된다.

### 주의점

- 변경 감지 기능
    - 원하는 속성만 선택해서 변경할 수 있다.
- 병합 기능
    - 모든 속성이 변경되므로 null도 같이 업데이트 된다.

```java

@Repository
@RequiredArgsConstructor
public class ItemRepository {

  public void save(Item item) {
    if (item.getId() == null) {
      em.persist(item);
    } else {
      em.merge(item);
    }
  }
}
```

- 식별자 값이 없는 null일때만 새로운 Entity로 판단해서 영속화하고 식별자가 있으면 병합하도록 구현한다.
- 준영속 상태인 상품 Entity를 수정할 때는 id 값이 있으므로 병합을 수행한다.

save()도 결국 변경된 데이터의 저장이라는 의미를 포함한다. 코드 상에서 식별자 유무에 따라 사용하면 저장과 수정을 구분하지 않아도 되므로 클라이언트 로직이 단순해진다.

참고로, `@GenreatedValue`에 의해 id가 자동으로 생성되는 상황이 아니라면 save()를 호출했을 때 식별자가 없다는 예외가 발생할 것이다.

### 가장 좋은 해결 방법

Entity를 변경할 때는 항상 변경 감지를 사용한다.

{% tabs %} {% tab title="잘못된 사례" %}

```java

@Controller
@RequiredArgsConstructor
public class ItemController {

  @PostMapping(value = "/items/{itemId}/edit")
  public String updateItem(@ModelAttribute("form") BookForm form) {
    Book book = new Book();

    book.setId(form.getId());
    book.setName(form.getName());
    book.setPrice(form.getPrice());
    book.setStockQuantity(form.getStockQuantity());
    book.setAuthor(form.getAuthor());
    book.setIsbn(form.getIsbn());

    itemService.saveItem(book);

    return "redirect:/items";
  }
}
```

{% endtab %} {% tab title="권장 코드" %}

```java
@Controller
@RequiredArgsConstructor
public class ItemController {

  @PostMapping(value = "/items/{itemId}/edit")
  public String updateItem(@ModelAttribute("form") BookForm form) {
    itemService.updateItem(form.getId(), form.getName(), form.getPrice());

    return "redirect:/items";
  }
}

@Service
@RequiredArgsConstructor
public class ItemService {

  private final ItemRepository itemRepository;

  /**
   * 영속성 컨텍스트가 자동 변경
   */
  @Transactional
  public void updateItem(Long id, String name, int price) {
    Item item = itemRepository.findOne(id);
    item.setName(name);
    item.setPrice(price);
  }
}
```

{% endtab %} {% endtabs %}

- 컨트롤러에서 어설프게 Entity를 생성하지 않는다.
- 트랜잭션이 있는 서비스 계층에 식별자와 변경할 데이터를 명확하게 전달한다.
- 트랜잭션이 있는 서비스 계층에서 영속 상태의 Entity를 조회하고 Entity 데이터를 직접 변경한다.