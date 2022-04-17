# 변경 감지와 병합

## 준영속 Entity

- JPA 영속성 컨텍스트가 더 이상 관리하지 않는 Entity
- 이미 DB에 한 번 저장되어 식별자가 있는 데이터

{% tabs %} {% tab title="ItemController.java" %}

```java

@Controller
@RequiredArgsConstructor
public class ItemController {

    @PostMapping(value = "/items/{itemId}/edit")
    public String updateItem(@ModelAttribute("form") BookForm form) {
        // 준영속 엔티티
        // 그냥 객체만 새로 만든 것이기 때문에 JPA가 관리하지 않는다.
        Book book = new Book();

        // id가 있으니 DB를 한 번 거쳤던 준영속 엔티티라는 걸 알 수 있다.
        book.setId(form.getId());
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        // itemRepository.save()를 호출해 병합 기능을 수행한다.
        itemService.saveItem(book);

        return "redirect:/items";
    }
}
```

{% endtab %} {% tab title="ItemRepository.java" %}

```java

@Repository
@RequiredArgsConstructor
public class ItemRepository {

    private final EntityManager em;

    public void save(Item item) {
        if (item.getId() == null) {
            // 없으면 신규로 등록한다.
            em.persist(item);
        } else {
            // 업데이트와 비슷하다고 일단 알아두고 나중에 설명한다.
            em.merge(item);
        }
    }
}
```

{% endtab %} {% endtabs %}

- 영속 상태 Entity
    - JPA가 관리하기 때문에 더티 체킹이 일어난다.
- 준영속 Entity
    - JPA가 관리하지 않으므로 DB 업데이트가 일어나지 않는다.
    - 변경 감지 기능과 병합 기능으로 수정할 수 있다.

## 변경 감지 기능

```java

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class ItemService {

    @Transactional
    public void updateItem(Long itemId, Item param) {
        // 영속 상태의 엔티티를 먼저 찾아온다.
        Item findItem = itemRepository.findOne(itemId);
        // 변경한다.
        findItem.setPrice(param.getPrice());
        findItem.setName(param.getName());
    }
}
```

1. 트랜잭션 안에서 Entity를 다시 조회한다.
2. 값을 변경한다.
3. 트랜잭션 커밋 시점에 변경 감지(Dirty Checking)가 일어난다.
    - commit() 후 flush() 할 때 바뀐 것을 찾아낸다.
4. 데이터베이스에 UPDATE SQL 실행된다.

준영속 Entity 값을 영속 상태 Entity에 세팅한다. 영속 상태 Entity는 더티체킹 덕분에 save()할 필요가 없다.

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

1. merge()가 준영속 Entity의 식별자 값으로 영속 Entity를 조회한다.
2. 준영속 Entity의 값으로 영속 Entity의 값을 모두 교체(병합)한다.
3. 트랜잭션 커밋 시점에 변경 감지 기능이 동작해서 데이터베이스에 UPDATE SQL이 실행된다.

영속성 컨텍스트에서 똑같은 식별자를 가진 데이터를 찾고, 파라미터 값으로 모든 값을 바꿔치기 한다.

### 동작 방식

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/07/screenshot%202021-05-22%20오후%207.50.15.png)

1. merge()를 실행한다.
2. 파라미터로 넘어온 준영속 Entity의 식별자 값으로 1차 캐시에서 Entity를 조회한다.
    - 만약 1차 캐시에 Entity가 없으면 데이터베이스에서 Entity를 조회하고, 1차 캐시에 저장한다.
3. 조회한 영속 Entity mergeMember에 member Entity의 값을 채워 넣는다.
    - member Entity의 모든 값을 mergeMember에 밀어 넣는다.
    - 이때 mergeMember의 `회원1`이라는 이름이 `회원명변경`으로 바뀐다.
4. 영속 상태인 mergeMember를 반환한다.

주의할 점은 파라미터로 넘어온 member는 영속 Entity로 안 바뀐다는 것이다. 병합이 된 mergeMember만 영속성 컨텍스트에서 관리된다.

### 주의점

- 변경 감지 기능
    - 원하는 속성만 선택해서 변경할 수 있다.
    - 병합 기능의 부작용 때문에 귀찮더라도 이 방법을 사용하는 게 좋다.
- 병합 기능
    - 모든 속성이 변경된다.
    - 넘어온 파라미터에 null이 있으면 같이 업데이트 된다.

### 해결 방법

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

- 식별자 유무에 따라 사용하면 저장과 수정을 구분하지 않아도 되므로 클라이언트 로직이 단순해진다.
    - 식별자 값이 null일때만 새로운 Entity로 판단해서 영속화한다.
    - 식별자가 있으면 병합한다.
- 식별자가 없는 데이터에 save()를 호출하면 식별자가 없다는 예외가 발생한다.

### 가장 좋은 해결 방법

- Entity를 변경할 때는 항상 변경 감지를 사용한다.

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
    - 잘못된 사례처럼 객체를 컨트롤러에서 만들어 넘기지 않는다.
    - 트랜잭션이 있는 서비스 계층에 식별자와 변경할 데이터를 명확하게 전달한다.
- 트랜잭션이 있는 서비스 계층에서 영속 상태의 Entity를 조회하고 Entity 데이터를 직접 변경한다.